<template>
  <div class="main-contain relative">
    <div class="header-contain flex flex-row justify-center align-middle text-center">
      <img :src="campaignInfo.store_logo" alt="" />
    </div>
    <div class="products-contain" v-if="isLoading">
      <ProgressSpinner />
    </div>

    <template v-else>
      <div v-if="route.query.id && route.query.message">
        <div class="alert alert-warning text-center">{{ route.query.message }}</div>
      </div>
      <div v-if="!campaignInfo.id" class="alert alert-danger text-center">
        الحملة غير موجودة
      </div>
      <template v-else>
        <div class="products-contain">
          <div class="first-section pb-4 flex flex-col gap-3">
            <div
              class="min-h-32  image-contain mt-3  flex justify-center align-middle text-center"
            >
              <Swiper
                v-if="selectedCardData && selectedCardData.slider.length > 0"
                :slides-per-view="1"
                space-between="0"
                :loop="true"
          
                class="mySwiper custom-swiper w-full h-full  "
       
              >
                <SwiperSlide
                  v-for="(card, index) in selectedCardData.slider"
                  :key="index"
                  class="swiper-slide image-container"
                >
                  <img :src="card" class="mx-auto image-width " alt="" />
                </SwiperSlide>
              </Swiper>
              <div
                v-if="selectedCardData && selectedCardData.slider.length === 0"
              class="image-container">
                <img
              
                :src="selectedCardData?.cover"
                class="mx-auto image-width "
                alt=""
              />
              </div>
  
              <div class="h-28 flex" v-if="!selectedCardData">
                <h2 class="my-auto font-bold">إختر منتج أولا</h2>
              </div>
            </div>
            <div v-if="selectedCardData" class="flex flex-row justify-center">
              <h2 class="font-almarai text-[8px] font-bold leading-[9.6px] text-center">
                {{ selectedCardData.name_ar }}
              </h2>
            </div>
          </div>
          <Swiper
            :slides-per-view="2.4"
            space-between="0"
            centered-slides="true"
            :breakpoints="{
              320: {
                slidesPerView: 1.5,
                spaceBetween: 0,
              },
              419: {
                slidesPerView: 2.3,
                spaceBetween: 5,
              },
              768: {
                slidesPerView: 2,
                spaceBetween: 0,
              },
              1024: {
                slidesPerView: 2.5,
                spaceBetween: 0,
              },
            }"
            class="mySwiper h-fit  my-6"
            loop="true"
          >
            <SwiperSlide v-for="card in cards" :key="card.id" class="swiper-slide card-slider">
              <ProductCard
                :card="card"
                :selected="selectedCardId"
                @click="selectCard(card)"
              />
            </SwiperSlide>
          </Swiper>
          <div class="px-[20px]" v-if="selectedCardData">
            <div>
              <div class="flex border-b">
                <button
                  v-for="(tab, index) in tabs"
                  :key="index"
                  @click="selectTab(index)"
                  :class="[
                    'px-4 py-2 cursor-pointer',
                    currentTab === index
                      ? 'border-b-2 border-black text-black font-bold'
                      : 'border-b-2 border-white font-semibold text-gray-600',
                  ]"
                >
                  {{ tab.title }}
                </button>
              </div>
              <div class="tab-content mt-3">
                <div v-if="currentTab === 0" class="details-content">
                  <ProductFirstTap :selected="selectedCardData" />
                </div>
                <div v-if="currentTab === 1" class="details-content">
                  <h2 class="font-almarai text-sm font-bold leading-4 text-right">
                    تعليقات المستتخدمين
                  </h2>
                  <div
                    class="mt-4 flex justify-center bg-gray-200 rounded-md py-4"
                    v-if="userComments.length === 0"
                  >
                    <h2 class="text-lg text-black text-bold">لا يوجد تعليقات</h2>
                  </div>
                  <div
                    v-else
                    class="min-h-56 mx-auto w-[95%] p-3 mt-3 rounded-lg bg-gray-200"
                  >
                    <div class="flex flex-row justify-between">
                      <h2
                        class="text-gray-500 my-auto font-almarai text-xs font-bold leading-3"
                      >
                        {{ userComments.length }} تعليق
                      </h2>
                      <button
                        @click="toggleSort"
                        class="flex flex-row-reverse justify-center gap-1 !bg-black !rounded-md !text-white font-almarai text-[10px] !p-2 font-light"
                      >
                        <Icon
                          name="mage:filter"
                          class="!text-white !text-[16px] my-auto"
                        />
                        {{ sortButtonText }}
                      </button>
                    </div>

                    <div class="mt-6 flex flex-col">
                      <div
                        v-for="(comment, index) in displayedComments"
                        :key="index"
                        class="mb-4"
                      >
                        <div class="flex flex-row justify-between">
                          <div class="flex flex-row gap-2">
                            <Avatar
                              image="https://primefaces.org/cdn/primevue/images/avatar/amyelsner.png"
                              class="!w-8 !h-8"
                              shape="circle"
                            />
                            <div class="my-auto flex flex-col gap-1">
                              <h2
                                class="font-almarai text-[10px] font-bold leading-[12px]"
                              >
                                {{ comment.client_name }}
                              </h2>
                              <div class="flex justify-center">
                                <Rating
                                  v-model="comment.rating"
                                  readonly
                                  class="text-[rgba(255,228,0,1)] text-[7px] my-auto"
                                />
                              </div>
                            </div>
                          </div>
                          <div class="my-auto">
                            <h2 class="flex flex-row-reverse gap-1">
                              <span
                                class="text-gray-600 font-almarai text-xs font-normal leading-2 tracking-tight text-right"
                              >
                                قام بالشراء , تم التقييم
                              </span>
                              <Icon
                                name="material-symbols:check-circle"
                                class="!text-[16px] !text-blue-400 !border-0 my-auto"
                              />
                            </h2>
                          </div>
                          <div class="my-auto">
                            <h2
                              class="text-gray-600 font-almarai text-xs font-normal leading-2 tracking-tight text-left"
                            >
                              {{ comment.created_at }}
                            </h2>
                          </div>
                        </div>
                        <div>
                          <h2
                            class="mt-3 text-[rgba(140,140,140,1)] font-almarai text-[10px] font-normal leading-[12px] tracking-[0.0024em] text-right"
                          >
                            {{ comment.comment }}
                          </h2>
                        </div>
                      </div>

                      <div v-if="!showLoadMoreButton" class="flex justify-center my-3">
                        <button
                          @click="loadMoreComments"
                          class="bg-black p-2 rounded-md text-white font-almarai text-xs font-bold leading-2 tracking-tight text-center"
                        >
                          تحميل المزيد
                        </button>
                      </div>
                    </div>
                  </div>
                  <h2 class="font-almarai text-sm font-bold leading-4 text-right mt-5">
                    مراجعات المستخدمين
                  </h2>
                  <div
                    class="mt-4 flex justify-center bg-gray-200 rounded-md py-4"
                    v-if="usersReviews.length === 0"
                  >
                    <h2 class="text-lg text-black text-bold">لا يوجد مراجعات</h2>
                  </div>
                  <div v-else>
                    <!-- {{ usersReviews }} -->
                    <Swiper
                      :slides-per-view="2"
                      space-between="10"
                      :breakpoints="{
                        320: {
                          slidesPerView: 1.5,
                          spaceBetween: 5,
                        },
                        768: {
                          slidesPerView: 2,
                          spaceBetween: 13,
                        },
                        1024: {
                          slidesPerView: 2,
                          spaceBetween: 10,
                        },
                      }"
                      class="mySwiper"
                      loop="true"
                    >
                      <SwiperSlide
                        v-for="card in usersReviews"
                        :key="card.id"
                        class="swiper-slide-reviews"
                      >
                        <div class="group !w-[93%] !h-full rounded-lg relative mt-3">
                          <div class="relative !h-full !w-full !rounded-lg overflow-hidden group">
                            <canvas 
                              ref="canvas" 
                              class="thumbnail-canvas absolute inset-0 transition-opacity duration-300 group-hover:opacity-0" 
                              style="display: block;"
                            ></canvas>
                            <video
                              class="video !h-full !w-full !rounded-lg overflow-hidden"
                              controls
                              :id="card.id"
                              v-if="card.video || (card.video && card.image)"
                              loop
                              ref="video"
                              @loadeddata="drawCanvas"
                            >
                              <source :src="card.video" type="video/mp4" />
                              Your browser does not support the video tag.
                            </video>
                          </div>
                          <img
                              v-if="!(card.video || (card.video && card.image))"
                            :src="card.image"
                            :id="card.id"
                            class="video !h-full !w-full !rounded-lg overflow-hidden"
                            alt=""
                          />
                          <div
                            class="absolute top-0 bottom-0 right-0 left-0 !rounded-lg bg-gradient-custom overflow-hidden pointer-events-none group-hover:opacity-0 transition-opacity duration-300 mt-auto !h-[100%] w-[100%]"
                          >
                            <!-- Set fixed width -->
                            <div
                              class="flex flex-col gap-1 right-3 bottom-10 absolute !rounded-lg"
                            >
                              <div class="flex justify-start mt-52">
                                <Rating
                                  v-model="card.rating"
                                  readonly
                                  class="text-[rgba(255,228,0,1)] text-[7px] my-auto"
                                />
                              </div>
                              <h2
                                class="text-[rgba(161,161,161,1)] font-almarai text-[12px] font-bold leading-[14.4px]"
                              >
                                {{ card.client_name }}
                              </h2>
                            </div>
                          </div>
                        </div>
                      </SwiperSlide>
                    </Swiper>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="footer-contain fixed bottom-0 inset-x-0 z-10  !px-8 md:!px-5 py-2">
          <button
            class="btn-Buy bg-black shadow-lg py-2 text-white flex items-center justify-center rounded-lg font-almarai text-[12px] font-bold leading-[14.4px]"
            @click="toggleModal"
          >
            شراء
          </button>

          <div
            v-if="showModal"
            :class="[
              'fixed modal inset-0 z-50 flex items-end justify-center bg-gray-200 bg-opacity-20',
              modalAnimationClass,
            ]"
          >
            <!-- Modal Content -->
            <div
              class="!w-full md:!w-full rounded-t-[24px] modal-contain bg-white py-3 z-50"
              @click.stop
            >
              <!-- Modal content -->
              <div class="flex flex-row px-4 mb-3 justify-between">
                <div class="flex flex-row gap-3 items-center">
                  <img
                    class="w-11 h-11 rounded-full"
                    src="https://i.postimg.cc/k4Gn6KbL/Ellipse-1.png"
                    alt="Rounded avatar"
                  />
                  <div class="flex flex-col justify-start">
                    <h2
                      class="font-almarai text-black text-xs font-normal leading-4 text-right"
                    >
                      مرحبا
                    </h2>
                    <h2
                      class="font-almarai text-black text-xs font-normal leading-4 text-right"
                    >
                      مستخدمنا العزيز
                    </h2>
                  </div>
                </div>
                <Icon
                  name="material-symbols:close"
                  @click="toggleModal"
                  class="!text-black !text-[16px] my-auto !cursor-pointer"
                />
              </div>
              <div class="content-modal pt-1 px-6  pb-2">
                <div class="flex flex-row mb-4 justify-between">
                  <div class="flex flex-col gap-1">
                    <h2
                      class="text-black text-[14px] font-bold leading-4 text-right font-almarai"
                    >
                      الإجمالى
                    </h2>
                    <h2
                      class="text-red-400 text-xs font-normal leading-3 text-right font-almarai"
                    >
                      لديك كوبون تخفيض؟
                    </h2>
                  </div>
                  <div>
                    <h2
                      class="text-black text-sm font-bold leading-5 text-right font-almarai"
                    >
                      {{ totalOrderPrice }} ر.س
                    </h2>
                  </div>
                </div>
                <div class="px-2">
                  <div
                    class="flex justify-center items-center py-2 gap-2 bg-gray-100"
                    :class="{ 'rounded-t-lg': isActive, 'rounded-lg': !isActive }"
                    @click="toggleAccordion"
                  >
                    <h3
                      class="font-almarai text-[12px] font-bold leading-[14.4px] text-right"
                    >
                      تفاصيل الفاتورة
                    </h3>
                    <Icon
                      name="material-symbols:keyboard-arrow-down"
                      :class="{ 'transform rotate-180': isActive }"
                      class="!text-black !text-[20px] my-auto"
                    />
                  </div>
                  <transition name="fade">
                    <div class="!px-3 bg-gray-100" :class="{ 'rounded-b-lg': isActive }">
                      <div v-show="isActive" class="py-4 border-y-[0.6px] border-gray-50">
                        <div class="flex flex-col gap-3">
                          <div class="grid grid-cols-2">
                            <div>
                              <h2
                                class="text-black font-almarai text-[12px] font-bold leading-[14.4px]"
                              >
                                {{ selectedCardData.name_ar }}
                              </h2>
                            </div>
                            <div class="flex justify-end">
                              <h2
                                class="text-black font-almarai text-[12px] font-bold leading-[14.4px]"
                              >
                                {{ selectedCardData.price }} ر.س
                              </h2>
                            </div>
                          </div>
                          <div
                            class="bundles-selected"
                            v-for="bundle in selectedBundles"
                            :key="bundle.id"
                          >
                            <div class="grid grid-cols-4 bundle">
                              <div class="col-span-2 flex justify-start items-center" >
                                <h2
                                  class="text-black font-almarai text-[12px] font-bold leading-[14.4px]"
                                >
                                  {{ bundle.name_ar }}
                                </h2>
                              </div>
                              <div class="flex justify-center  items-center">
              
                                <button
                                @click="removeBundle(bundle.id)"
                                class="text-black  mt-1"
                                aria-label="Remove Bundle"
                              >
                                <!-- Replace with your desired icon -->
                                <Icon name="material-symbols:cancel-rounded" />
                              </button>
                        
                              </div>

                              <div class="flex justify-end  items-center">
              
                                <h2
                                class="text-black font-almarai text-[12px] font-bold leading-[14.4px] mr-2"
                              >
                                {{ bundle.price }} ر.س
                              </h2>
                        
                              </div>
   
                            </div>
                          </div>

                          <div
                            v-if="selectedGift.name_ar && !isDisabled"
                            class="grid grid-cols-4 gift"
                          >
                            <div class="col-span-2 flex justify-start items-center">
                              <h2
                                class="text-black font-almarai text-[12px] font-bold leading-[14.4px]"
                              >
                                {{ selectedGift.name_ar }}
                              </h2>
                            </div>
                            <div class="flex justify-center  items-center">
              
                              <button
                              @click="removeGift()"
                              class="text-black  mt-1"
                              aria-label="Remove Bundle"
                            >
                              <!-- Replace with your desired icon -->
                              <Icon name="material-symbols:cancel-rounded" />
                            </button>
                      
                            </div>
                            <div class="flex justify-end gap-1  items-center">
                              <h2
                                v-if="     campaignInfo.discount_type !== 'percentage' &&
                                selectedGift.id !== 'has_discount' "
                                class="text-black font-almarai text-[9px] md:text-[10px] font-bold leading-[14.4px]"
                              >
                                {{ selectedGift.price }}
                                <span
                                  v-if="
                                    campaignInfo.discount_type !== 'percentage' &&
                                    selectedGift.id !== 'has_discount'
                                  "
                                  >ر.س</span
                                >
                              </h2>
                              <h2
                                v-if="     campaignInfo.discount_type !== 'percentage' &&
                                selectedGift.id !== 'has_discount' "
                                class="line-through text-gray-500 font-almarai text-[9px] md:text-[10px] font-bold leading-[14.4px]"
                              >
                                {{ selectedGift.price_before_discount }}
                                <span
                                  v-if="
                                    campaignInfo.discount_type !== 'percentage' &&
                                    selectedGift.id !== 'has_discount'
                                  "
                                  >ر.س</span
                                >
                              </h2>
                            </div>
                          </div>
                        </div>
                      </div>
                      <div v-show="isActive">
                        <div
                          v-if="showDelivery && selectedGift.id !== 'has_free_shipping'"
                          class="flex flex-row justify-between pt-1 pb-[10px]"
                        >
                          <h2
                            class="text-black font-almarai text-[12px] font-bold leading-[14.4px] text-right"
                          >
                            التوصيل
                          </h2>
                          <h2
                            class="text-black font-almarai text-[12px] font-bold leading-[14.4px] text-right"
                          >
                            {{ selectedCityData.price }} ر.س
                          </h2>
                        </div>
                      </div>
                    </div>
                  </transition>
                </div>

                <div class="delivery-info mt-3">
                  <h2
                    class="text-black font-almarai text-[14px] font-bold leading-[16.8px] text-right"
                  >
                    بيانات التوصيل
                  </h2>
                  <div class="mt-2">
                    <label
                      for="fullNameInput"
                      class="form-label text-black font-almarai text-[12px] font-bold leading-[14.4px] text-right !mb-3"
                    >
                      الاسم الثلاثى
                    </label>
                    <input
                      type="text"
                      class="form-control"
                      id="fullNameInput"
                      v-model="form.fullName"
                      placeholder="الاسم الثلاثى"
                    />
                    <div v-if="errors.fullName" class="text-red-500 mt-2">
                      {{ errors.fullName }}
                    </div>
                  </div>
                  <div class="mt-3">
                    <label
                      for="phoneNumberInput"
                      class="form-label text-black font-almarai text-[12px] font-bold leading-[14.4px] text-right !mb-3"
                    >
                      تأكيد رقم الجوال
                    </label>
                    <div class="flex  mb-3">
                      <button
                        class="bg-black text-white py-[9px] px-[24px] !rounded-r-[4px] rounded-l-[0px] font-almarai text-[12px] font-bold leading-[14.4px] flex items-center justify-center"
                        type="button"
                        id="button-addon1"
                        @click="verifyPhone"
                        :disabled="isLoadingPhone || otpConfirmed"
                      >
                        <template v-if="isLoadingPhone">
                          <!-- EOS loading icon from nuxt-icon -->
                          <Icon name="eos-icons:loading" class="animate-spin w-4 h-4" />
                        </template>
                        <template v-else> تأكيد </template>
                      </button>
                 
                      <a-input-group     class="phone-input" compact>
                        <!-- <a-input v-model:value="value1"  readonly /> -->
                        <a-select v-model:value="value1" style="width: 25%" >
                          <a-select-option value="+966">+966</a-select-option>
                          <a-select-option value="+20">+20</a-select-option>
                        </a-select>
                        <a-input v-model:value="value2" style="width: 75%"   />
                      </a-input-group>
                    </div>
                    <div v-if="errorPhone" class="text-red-500 mt-2">
                      {{ errorPhone }}
                    </div>
                    <div v-if="errors.phoneNumber" class="text-red-500 mt-2">
                      {{ errors.phoneNumber }}
                    </div>
                    <div v-if="errors.otp" class="text-red-500 mt-2">
                      {{ errors.otp }}
                    </div>
                    <div v-if="otpConfirmed" class="text-green-400 mt-2">
                      تم تأكيد رقم الجوال
                    </div>
                    <div v-if="showOtpPopup">
                      <div class="overlay" @click="closeOtpPopup"></div>
                      <div class="otp-popup">
                        <div class="popup-content">
                          <h3
                            class="font-almarai text-[14px] font-bold leading-[16.8px] text-right mb-3"
                          >
                            ادخل رمز التحقق
                          </h3>
                          <div ref="otpCont" class="inputs-contain">
                            <!-- <input
                              type="text"
                              class="digit-box"
                              v-for="(el, ind) in digits"
                              :key="el + ind"
                              v-model="digits[ind]"
                              :autofocus="ind === 0"
                              maxlength="1"
                              @input="handleInput($event, ind)"
                              @keydown="handleBackspace($event, ind)"
                            /> -->
                          </div>
                          <v-otp-input
                          length="5"
                          class="otp-inputs"
                          v-model="otpCodeNumber"
                        ></v-otp-input>
                          <div v-if="errorVerify" class="text-lg text-red-500 mt-2">
                            {{ errorVerify }}
                          </div>
                          <div class="flex w-full">
                            <button
                              @click="submitOtp"
                              class="bg-black rounded-lg w-full text-white py-2 px-4 mt-4 font-almarai text-[14px] font-bold leading-[16.8px]"
                            >
                              تأكيد
                            </button>
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>
                  <div class="mt-3">
                    <label
                      for="citySelect"
                      class="form-label text-black font-almarai text-[12px] font-bold leading-[14.4px] text-right !mb-3"
                    >
                      المدينه
                    </label>
                    <a-select
                    v-model:value="form.selectedCity"
                    placeholder="المدينه"
                    show-search
                    :options="cities"
                    :filter-option="filterOption"
                    @focus="handleFocus"
                    @blur="handleBlur"
                    @change="handleChange"
                    style="width: 100%;"
                  >
                  </a-select>
                    <div v-if="errors.selectedCity" class="text-red-500 mt-2">
                      {{ errors.selectedCity }}
                    </div>
                  </div>
                  <div class="mt-3">
                    <label
                      for="placeInput"
                      class="form-label text-black font-almarai text-[12px] font-bold leading-[14.4px] text-right !mb-3"
                    >
                      الحى ووصف المكان
                    </label>
                    <input
                      type="text"
                      class="form-control"
                      id="placeInput"
                      v-model="form.place"
                      placeholder="الحى"
                    />
                    <div v-if="errors.place" class="text-red-500 mt-2">
                      {{ errors.place }}
                    </div>
                  </div>

                  <div class="mt-4">
                    <h2
                      class="text-black font-almarai text-[14px] font-bold leading-[14.4px]"
                    >
                      قم باختيار عرض
                    </h2>
                    <div class="mt-3">
                      <div
                        class="mt-4 flex justify-center bg-gray-200 rounded-md py-4"
                        v-if="bundles.length === 0"
                      >
                        <h2 class="text-lg text-black text-bold">لا يوجد عروض</h2>
                      </div>
                      <div v-else>
                        <Swiper
                          :slides-per-view="3"
                          space-between="15"
                          :breakpoints="{
                            320: {
                              slidesPerView: 2,
                              spaceBetween: 10,
                            },
                            419: {
                              slidesPerView: 2.2,
                              spaceBetween: 20,
                            },
                            768: {
                              slidesPerView: 3,
                              spaceBetween: 15,
                            },
                            1024: {
                              slidesPerView: 2.5,
                              spaceBetween: 15,
                            },
                          }"
                          class="mySwiper !p-0 md:!p-3"
                          loop="true"
                        >
                          <SwiperSlide
                            v-for="bundle in bundles"
                            :key="bundle.id"
                            class="!h-40"
                          >
                            <Card
                              class="cursor-pointer p-1 !w-full !h-[100%] !rounded-lg transition-all duration-300 shadow-sm hover:shadow-lg"
                              :class="{
                                'border-2 border-black card-shadow': selectedBundles.some(
                                  (item) => item.id === bundle.id
                                ),
                                'border-2 border-white card-shadow': !selectedBundles.some(
                                  (item) => item.id === bundle.id
                                ),
                              }"
                              @click="toggleSelectBundle(bundle)"
                            >
                              <template #header>
                                <div class="w-full h-20 mb-2 card-image relative">
                                  <img
                                    class="w-full mx-auto object-contain"
                                    alt="product image"
                                    :src="bundle.cover"
                                  />
                                </div>
                              </template>
                              <template #title>
                                <div
                                  class="text-black !h-10 items-center font-almarai text-[12px] font-normal leading-[14.4px] tracking[-0.02em]"
                                >
                                  <h2>{{ bundle.name_ar }}</h2>
                                </div>
                              </template>
                              <template #content>
                                <div class="flex flex-row gap-2 mt-1 !h-4">
                                  <p
                                    class="font-almarai text-[14px] font-bold leading-[14.4px]"
                                  >
                                    {{ bundle.price }}
                                    <span
                                      class="font-almarai text-[10px] font-bold leading-[9.6px]"
                                      >ر.س</span
                                    >
                                  </p>
                                  <p
                                    v-if="bundle.price_before_discount"
                                    class="font-almarai text-[12px] font-medium leading-[14.4px] line-through text-gray-500"
                                  >
                                    {{ bundle.price_before_discount }}
                                    <span
                                      class="font-almarai text-[8px] font-medium leading-[9.6px]"
                                      >ر.س</span
                                    >
                                  </p>
                                </div>
                              </template>
                            </Card>
                          </SwiperSlide>
                        </Swiper>
                      </div>
                      <div class="w-full mt-3 bundle-popup !px-2 md:!px-5">
                        <button
                          class="bundle-popup-button w-full bg-black py-2 text-white flex items-center justify-center rounded-lg font-almarai text-[14px] font-bold leading-[14.4px]"
                          @click="togglePopup"
                        >
                          اختر عرض من العروض
                        </button>
                      </div>

                      <div
                        v-if="isPopupVisible"
                        class="fixed inset-0 bg-black bg-opacity-50 flex justify-center items-center z-[1000]"
                      >
                        <div
                          class="bg-white px-5 py-4 md:!px-8 md:!py-6 rounded-3xl w-[90%] md:w-[85%] max-h-[90%] overflow-y-auto"
                        >
                          <!-- Swiper for Bundles inside the Popup -->
                          <div class="flex flex-row justify-start">
                            <Icon
                              @click="togglePopup"
                              name="material-symbols:close-rounded"
                              class="!text-black text-[16px] md:!text-[14px] cursor-pointer"
                            />
                          </div>
                          <div class="max-h-[350px] overflow-y-auto">
                            <div
                              class="flex flex-col gap-4"
                              v-for="bundle in bundles"
                              :key="bundle.id"
                            >
                              <div
                                class="form-check form-check-reverse rounded-md flex items-center justify-center !my-2"
                                :class="{
                                  'card-shadow': selectedBundles.some(
                                    (item) => item.id === bundle.id
                                  ),
                                  'card-shadow-inactive': !selectedBundles.some(
                                    (item) => item.id === bundle.id
                                  ),
                                }"
                              >
                                <!-- Checkbox aligned to the right -->
                                <input
                                  class="form-check-input cursor-pointer h-5 w-5 ml-2"
                                  type="checkbox"
                                  :id="'bundle_' + bundle.id"
                                  :checked="
                                    selectedBundles.some((item) => item.id === bundle.id)
                                  "
                                  @change="toggleSelectBundle(bundle)"
                                />
                                <label
                                  class="form-check-label w-full"
                                  :for="'bundle_' + bundle.id"
                                >
                                  <div class="cursor-pointer p-1 !w-[100%] !h-[100%]">
                                    <div class="flex flex-row gap-2">
                                      <div
                                        class="mb-2 h-20 w-20 bg-gray-200 flex justify-center items-center text-center relative overflow-hidden"
                                      >
                                        <img
                                          class="object-contain !h-full !w-full"
                                          alt="product image"
                                          :src="bundle.cover"
                                        />
                                      </div>
                                      <div class="flex flex-col justify-center">
                                        <div
                                          class="text-black !h-8 items-center font-almarai text-[12px] font-normal leading-[14.4px] tracking-[-0.02em]"
                                        >
                                          <h2>{{ bundle.name_ar }}</h2>
                                        </div>
                                        <div class="flex flex-row gap-2 !h-4">
                                          <p
                                            class="font-almarai text-[12px] font-bold leading-[14.4px]"
                                          >
                                            {{ bundle.price }}
                                            <span
                                              class="font-almarai text-[10px] font-bold leading-[9.6px]"
                                              >ر.س</span
                                            >
                                          </p>
                                          <p
                                            v-if="bundle.price_before_discount"
                                            class="font-almarai text-[10px] font-medium leading-[14.4px] text-gray-500"
                                          >
                                            <span class="line-through">
                                              {{ bundle.price_before_discount }}
                                            </span>

                                            <span
                                              class="font-almarai text-[8px] font-medium leading-[9.6px]"
                                              >ر.س</span
                                            >
                                          </p>
                                        </div>
                                      </div>
                                    </div>
                                  </div>
                                </label>
                                <!-- Input aligned to the right and vertically centered -->
                              </div>
                            </div>
                          </div>
                          <div
                            class="flex flex-col gap-2 mt-2 px-2 md:!px-5 justify-center"
                          >
                            <h2
                              class="text-black font-almarai text-sm font-bold leading-5 text-center"
                            >
                              الاجمالى : {{ totalPrice }} ر.س
                            </h2>
                            <button
                              class="bg-black shadow-md text-white border-none py-2 px-2.5 cursor-pointer rounded-lg"
                              @click="togglePopup"
                            >
                              اضف الى المحفظه
                            </button>
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>
                  <div class="mt-4 !pb-14">
                    <h2
                      class="text-black font-almarai text-[14px] font-bold leading-[14.4px] text-right"
                    >
                      اضف العرض الى المحفظه واحصل على هدية مجانيه
                    </h2>
                    <div class="mt-3">
                      <div>
                        <Swiper
                          :slides-per-view="2.5"
                          space-between="10"
                          :breakpoints="{
                            320: {
                              slidesPerView: 2,
                              spaceBetween: 5,
                            },
                            768: {
                              slidesPerView: 2.5,
                              spaceBetween: 10,
                            },
                            1024: {
                              slidesPerView: 2.5,
                              spaceBetween: 10,
                            },
                          }"
                          class="mySwiper !p-0 md:!p-3"
                          loop="true"
                        >
                          <SwiperSlide v-for="gift in gifts" :key="gift.id" class="!h-44">
                            <Card
                              class="cursor-pointer p-2 h-full !w-full !rounded-lg transition-all duration-300 hover:shadow-lg"
                              :class="{
                                'border-2 border-black':
                                  selectedGift.id === gift.id || !isDisabled,
                                'border-2 border-gray-200':
                                  selectedGift.id !== gift.id || isDisabled,
                              }"
                              @click="toggleSelectGift(gift)"
                            >
                              <template #header>
                                <div class="w-full h-24 mb-2 card-image relative">
                                  <img
                                    class="object-contain h-full mx-auto"
                                    alt="product image"
                                    :src="gift.image"
                                  />
                                  <div
                                    v-if="isDisabled"
                                    class="absolute inset-0 bg-black bg-opacity-50"
                                  >
                                    <div
                                      class="absolute top-1 right-2 h-4 md:!h-6 w-4 md:!w-6 rounded-full bg-gray-200 bg-opacity-50 flex items-center justify-center"
                                    >
                                      <Icon
                                        name="material-symbols:lock-outline-sharp"
                                        class="!text-white text-[10px] md:!text-[14px]"
                                      />
                                    </div>
                                  </div>
                                </div>
                              </template>
                              <template #title>
                                <div class="flex flex-col gap-2">
                                  <div
                                    class="text-black items-center font-almarai text-[14px] font-medium leading-[14.4px] tracking[-0.02em]"
                                  >
                                    <h2>{{ gift.name_ar }}</h2>
                                  </div>
                                  <div class="flex flex-row gap-2 mt-1 !h-4">
                                    <p
                                      class="font-almarai text-[14px] font-bold leading-[14.4px]"
                                    >
                                      <span v-if="gift.price !== '0.00'">{{
                                        gift.price
                                      }}</span>
                                      <span v-if="gift.price === '0.00'">مجانى</span>
                                      <span
                                        v-if="gift.price !== '0.00'"
                                        class="font-almarai text-[10px] font-bold leading-[9.6px]"
                                        >ر.س</span
                                      >
                                    </p>
                                    <p
                                      v-if="gift.price_before_discount"
                                      class="font-almarai text-[12px] font-medium leading-[14.4px] line-through text-gray-500"
                                    >
                                      {{ gift.price_before_discount }}
                                      <span
                                        class="font-almarai text-[8px] font-medium leading-[9.6px]"
                                        >ر.س</span
                                      >
                                    </p>
                                  </div>
                                </div>
                              </template>
                              <template #content> </template>
                            </Card>
                          </SwiperSlide>

                          <SwiperSlide
                            v-if="campaignInfo.has_free_shipping === 1"
                            class="!h-44"
                          >
                            <Card
                              class="cursor-pointer p-2 h-full !w-full !rounded-lg transition-all duration-300 hover:shadow-lg"
                              :class="{
                                'border-2 border-black':
                                  selectedGift.id === freeShipping.id || !isDisabled,
                                'border-2 border-gray-200':
                                  selectedGift.id !== freeShipping.id || isDisabled,
                              }"
                              @click="toggleSelectGift(freeShipping)"
                            >
                              <template #header>
                                <div class="w-full h-24 mb-2 card-image relative">
                                  <img
                                    class="object-contain h-full mx-auto"
                                    alt="product image"
                                    :src="freeShipping.image"
                                  />
                                  <div
                                    v-if="isDisabled"
                                    class="absolute inset-0 bg-black bg-opacity-50"
                                  >
                                    <div
                                      class="absolute top-1 right-2 h-4 md:!h-6 w-4 md:!w-6 rounded-full bg-gray-200 bg-opacity-50 flex items-center justify-center"
                                    >
                                      <Icon
                                        name="material-symbols:lock-outline-sharp"
                                        class="!text-white text-[10px] md:!text-[14px]"
                                      />
                                    </div>
                                  </div>
                                </div>
                              </template>
                              <template #title>
                                <div
                                  class="text-black items-center font-almarai text-[14px] font-medium leading-[14.4px] tracking[-0.02em]"
                                >
                                  <h2>{{ freeShipping.name_ar }}</h2>
                                </div>
                              </template>
                            </Card>
                          </SwiperSlide>
                          <SwiperSlide
                            v-if="campaignInfo.has_discount === 1"
                            class="!h-44"
                          >
                            <Card
                              class="cursor-pointer p-2 h-full !w-full !rounded-lg transition-all duration-300 hover:shadow-lg"
                              :class="{
                                'border-2 border-black':
                                  selectedGift.id === discountAmount.id || !isDisabled,
                                'border-2 border-gray-200':
                                  selectedGift.id !== discountAmount.id || isDisabled,
                              }"
                              @click="toggleSelectGift(discountAmount)"
                            >
                              <template #header>
                                <div class="w-full h-24 mb-2 card-image relative">
                                  <img
                                    class="object-contain h-full mx-auto"
                                    alt="product image"
                                    :src="discountAmount.image"
                                  />
                                  <div
                                    v-if="isDisabled"
                                    class="absolute inset-0 bg-black bg-opacity-50"
                                  >
                                    <div
                                      class="absolute top-1 right-2 h-4 md:!h-6 w-4 md:!w-6 rounded-full bg-gray-200 bg-opacity-50 flex items-center justify-center"
                                    >
                                      <Icon
                                        name="material-symbols:lock-outline-sharp"
                                        class="!text-white text-[10px] md:!text-[14px]"
                                      />
                                    </div>
                                  </div>
                                </div>
                              </template>
                              <template #title>
                                <div
                                  class="text-black items-center font-almarai text-[14px] font-medium leading-[14.4px] tracking[-0.02em]"
                                >
                                  <h2>{{ discountAmount.name }}</h2>
                                </div>
                              </template>
                            </Card>
                          </SwiperSlide>
                        </Swiper>
                      </div>
                    </div>
                  </div>
                  <div class="fixed bottom-0 inset-x-0 z-20 bg-white py-2 mt-4 !px-2 md:!px-5">
                    <div class="flex flex-col gap-2 justify-center w-full">
                      <div>
                        <button
                        class="w-full bg-black py-2 text-white flex items-center justify-center rounded-lg font-almarai text-[12px] font-bold leading-[14.4px]"
                        @click="submitForm"
                      >
                        اتمام الدفع
                      </button>
                      </div>
                      <div>
                        <img src="https://i.postimg.cc/sDGwfNmD/payment.png" alt="">
                      </div>

                    </div>
            
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </template>
    </template>
    <Toast :message="toastMessage" />
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

// import { useHead } from '@vueuse/head';
// import payment from "assets/images/payment.svg"
import Toast from "~/components/Toast.vue";
import "swiper/css";
import "swiper/css/navigation";
// Import required modules
import { Navigation } from "swiper/modules";

const modules = [Navigation];
const route = useRoute();
import { Swiper, SwiperSlide } from "swiper/vue";
import "swiper/swiper-bundle.css";
const tabs = ref([
  { title: "التفاصيل", value: "0" },
  { title: "التقييمات", value: "1" },
]);
const cards = ref([]);
const bundles = ref([]);
const gifts = ref([]);
const selectedCardId = ref(null);
const selectedCardData = ref(null);
const error = ref(null);
const isLoading = ref(true);
const userComments = ref([]);
const displayedComments = ref([]);
const usersReviews = ref([]);
const showLoadMoreButton = ref(true);
const campaignInfo = ref({});
const currentTab = ref(0);
const isActive = ref(false);
const selectedGift = ref({
  price: "0.00",
});
const selectedBundles = ref([]);
const isPopupVisible = ref(false);
const showModal = ref(false);
const modalAnimationClass = ref("modal-top");
const otpConfirmed = ref(false);
const showOtpPopup = ref(false);
const errorPhone = ref(null);
const isLoadingPhone = ref(false);
const errorVerify = ref(null);
const isLoadingVerify = ref(false);
const value1 = ref('+966');
const value2 = ref();
const freeShipping = ref({
  id: "has_free_shipping",
  name_ar: "توصيل مجانى",
  name_en: "Free Shipping",
  name: "Free Shipping",
  description: null,
  price: "0.00",
  price_before_discount: null,
  image: "https://i.postimg.cc/rwsv0w7m/image.png",
});
console.log(route.query);

const canvas = ref(null);
const video = ref(null);

// Draw the first frame of the video onto the canvas
const drawCanvas = () => {
  const canvasEl = canvas.value; // Assign the canvas element to a variable
  const videoEl = video.value; // Assign the video element to a variable

  if (canvasEl && videoEl) {
    const context = canvasEl.getContext('2d'); // Get the context of the canvas

    canvasEl.width = videoEl.videoWidth; // Set the canvas width to the video width
    canvasEl.height = videoEl.videoHeight; // Set the canvas height to the video height

    videoEl.currentTime = 0; // Set the current time to the beginning
    videoEl.addEventListener('loadeddata', () => {
      context.drawImage(videoEl, 0, 0, canvasEl.width, canvasEl.height); // Draw the image onto the canvas
    }, { once: true });
  }
};

// Ensure canvas reference is available on mount
onMounted(() => {
  // Call drawCanvas only if video is defined
  if (video.value) {
    drawCanvas();
  }
});
const cities = ref([]);
const form = reactive({
  fullName: "",
  phoneNumber: "",
  selectedCity: "",
  place: "",
});
watch(value2, (newValue) => {
  // Update form.phoneNumber by removing the '+' from value1 and appending value2
  form.phoneNumber = value1.value.replace('+', '') + newValue;
});
// Error state
const errors = reactive({
  fullName: null,
  phoneNumber: null,
  selectedCity: null,
  place: null,
  otp: null,
});
// Ref to hold the selected city object
const selectedCity = ref(null);
const selectedCityObject = ref(null);
const isDisabled = computed(() => selectedBundles.value.length === 0);

// Watch for changes in isDisabled
watch(isDisabled, (newValue) => {
  // If no bundles are selected (isDisabled becomes false)
  if (newValue === false) {
    // Reset selectedGift to an empty object
    selectedGift.value = {};
  }
});
// Method to toggle gift selection
function toggleSelectGift(gift) {
  // Prevent selection if selectedBundles is empty
  if (!isDisabled.value) {
    // If the selectedGift is already the one clicked, deselect it
    if (selectedGift.value && selectedGift.value.id === gift.id) {
      selectedGift.value = {};
    } else {
      // Select the new gift
      selectedGift.value = gift;
    }
  }
}

// Watch for changes in `showModal` and update the animation class
watch(showModal, (newVal) => {
  if (newVal) {
    modalAnimationClass.value = "modal-top"; // slideUp on open
  } else {
    modalAnimationClass.value = "modal-bottom"; // slideDown on close
  }
});
const toastMessage = ref("");

function showToast(message) {
  toastMessage.value = message;
  setTimeout(() => {
    toastMessage.value = "";
  }, 3000);
}

function toggleModal() {
  if (!selectedCardData.value) {
    showToast("اختر منتج أولا");
    return;
  }
  showModal.value = !showModal.value;
}
const digits = reactive(Array(5).fill(null)); // Initialize with 5 null values
const otpCodeNumber = ref()
const otpCont = ref(null);

// Handle input event to restrict to numbers and focus the next input if filled
const handleInput = (event, index) => {
  const value = event.target.value;

  // Only allow digits (0-9)
  if (/^\d*$/.test(value)) {
    digits[index] = value;
  } else {
    digits[index] = "";
  }

  // Automatically focus the next input field if the current one is filled
  if (digits[index] && index < digits.length - 1) {
    const nextInput = otpCont.value.children[index + 1];
    if (nextInput) {
      nextInput.focus();
    }
  }
};

// Handle backspace to remove value and move focus if needed// Handle backspace to remove value and move focus if needed
const handleBackspace = (event, index) => {
  // Check if the pressed key is Backspace
  if (event.key === "Backspace") {
    // Clear current input if it's not empty
    if (digits[index] !== "") {
      digits[index] = ""; // Clear current input
      event.preventDefault(); // Prevent the default backspace behavior (i.e., moving focus to previous)
    }
    // If current input is empty, move to the previous one
    else if (index > 0) {
      const prevInput = otpCont.value.children[index - 1];
      if (prevInput) {
        prevInput.focus();
        digits[index - 1] = ""; // Clear the previous input
        event.preventDefault(); // Prevent default behavior
      }
    }
  } else {
    // Allow only digits (0-9) and control keys (Backspace, Delete)
    if (!/^[0-9]$/.test(event.key) && event.key !== 'Backspace' && event.key !== 'Delete') {
      event.preventDefault(); // Prevent non-digit input
    }
  }
};
// Function to toggle selection
const toggleSelectBundle = (bundle) => {
  const index = selectedBundles.value.findIndex((item) => item.id === bundle.id);
  if (index === -1) {
    // If not selected, add it
    selectedBundles.value.push(bundle);
  } else {
    // If selected, remove it
    selectedBundles.value.splice(index, 1);
  }
};

const totalPrice = computed(() => {
  return selectedBundles.value.reduce((sum, item) => sum + item.price, 0);
});

const totalOrderPrice = computed(() => {
  // Calculate individual prices
  const cardPrice = parseFloat(selectedCardData.value.price) || 0;
  const bundlesPrice = selectedBundles.value.reduce((sum, item) => {
    return sum + (parseFloat(item.price) || 0);
  }, 0);

  // Calculate gift price only if id is not 'has_free_shipping' or 'has_discount'
  const giftPrice =
    selectedGift.value &&
    selectedGift.value.id !== "has_free_shipping" &&
    selectedGift.value.id !== "has_discount"
      ? parseFloat(selectedGift.value.price) || 0
      : 0;

  // Add city price only if id is not 'has_free_shipping'
  const cityPrice =
    selectedGift.value && selectedGift.value.id !== "has_free_shipping"
      ? parseFloat(selectedCityData.value?.price) || 0
      : 0;

  // Calculate total
  const total = cardPrice + bundlesPrice + giftPrice + cityPrice;
  return total;
});

// Function to toggle the popup visibility
const togglePopup = () => {
  isPopupVisible.value = !isPopupVisible.value;
};

//  accordion

const toggleAccordion = () => {
  isActive.value = !isActive.value;
};

function selectTab(index) {
  currentTab.value = index;
}

const verifyPhone = async () => {
  isLoadingPhone.value = true;
  errorPhone.value = null; // Reset error message
  console.log("form.selectedCity", form.selectedCity);

  try {
    const body = {
      whatsapp_number: form.phoneNumber,
    };

    const { data } = await useFetch("https://up.ft.sa/api/v1/campaign/verify/whatsapp", {
      method: "POST",
      body: body,
    });

    console.log("Verify", data);
    if (data.value?.status === true) {
      showOtpPopup.value = true;
    } else {
      errorPhone.value = data.value?.message ?? "الحقل رقم الواتساب مطلوب";
    }
  } catch (err) {
    errorPhone.value = err.message; // Set error message
  } finally {
    isLoadingPhone.value = false;
  }
};

const submitOtp = async () => {
  const otp = digits.join("");
  isLoadingVerify.value = true;

  errorVerify.value = null; // Reset error message

  try {
    const body = {
      whatsapp_number: form.phoneNumber,
      otp: otpCodeNumber.value,
    };

    const { data } = await useFetch("https://up.ft.sa/api/v1/campaign/verify/otp", {
      method: "POST",
      body: body,
      headers: {
        ACCEPT_LANGUAGE: "ar",
      },
    });

    console.log("Verify", data);

    // Assuming response success opens the OTP input
    if (data.value.status === true) {
      showOtpPopup.value = false;
      otpConfirmed.value = true;
      errors.otp = null;
    } else {
      errorVerify.value = data.value.message;
    }
  } catch (err) {
    errorVerify.value = err.message; // Set error message
  } finally {
    isLoadingVerify.value = false;
  }
  // Close the popup after submission
};
watch(
  otpCodeNumber,
  (newCode) => {
    if (newCode.length === 5 && /^\d+$/.test(newCode)) {
      submitOtp(); 
    }
  }
);
const setMetaTags = (campaign) => {
  useHead({
    title: campaign.meta_title || "Up Sale",
    meta: [
      {
        name: "description",
        content: campaign.meta_description || "Default Description",
      },
      { name: "keywords", content: campaign.meta_keywords || "Default Keywords" },
    ],
    link: [{ rel: "icon", href: campaign.store_logo }],
  });
};
const fetchCampaign = async () => {
  try {
    const response = await fetch(
      `https://up.ft.sa/api/v1/campaign/${route.params.store}/${route.params.uuid}`,
      {
        headers: {
          ACCEPT_LANGUAGE: "ar",
        },
      }
    );

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    if (data.campaign) {
      campaignInfo.value = data.campaign;
      setMetaTags(data.campaign);
    }
  } catch (err) {
    error.value = "Error fetching products: " + err.message;
    console.error("Error fetching products:", err);
  } finally {
    isLoading.value = false;
  }
};
const discountAmount = computed(() => ({
  id: "has_discount",
  name_ar: "خصم",
  name_en: "خصم",
  name:
    campaignInfo.value.discount_type === "fixed"
      ? `خصم بقيمة ${campaignInfo.value.discount_value} ر.س`
      : `خصم بقيمة ${campaignInfo.value.discount_value} %`,
  description: null,
  price:
    campaignInfo.value.discount_type === "fixed"
      ? `  ${campaignInfo.value.discount_value} ر.س`
      : `  ${campaignInfo.value.discount_value} %`,
  price_before_discount: null,
  image: "https://i.postimg.cc/L5rx0jmQ/discount.png",
}));
const fetchProducts = async () => {
  try {
    const response = await fetch(
      `https://up.ft.sa/api/v1/campaign/products/${route.params.uuid}`,
      {
        headers: {
          ACCEPT_LANGUAGE: "ar",
        },
      }
    );

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    if (data) {
      cards.value = data.data;
      selectedCardData.value = data?.data[0]
      selectedCardId.value = data?.data[0]?.id
      
    }
  } catch (err) {
    error.value = "Error fetching products: " + err.message;
    console.error("Error fetching products:", err);
  } finally {
    isLoading.value = false;
  }
};
const fetchCities = async () => {
  try {
           const response = await fetch(`https://up.ft.sa/api/v1/campaign/get-cities/${route.params.uuid}`, {
      headers: {
        'ACCEPT_LANGUAGE': 'ar',
      },
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    if (data) {
      console.log("cities" + data.data);

      cities.value = data.data.map((city) => ({
        value: city.id,
        label: city.name,
        price: city.price,
      }));
    }
  } catch (err) {
    error.value = "Error fetching products: " + err.message;
    console.error("Error fetching products:", err);
  } finally {
    isLoading.value = false;
  }
};
const showDelivery = ref(false)

const selectedCityData = computed(() => {
  return cities.value.find((city) => city.value === form.selectedCity) || {};
});

const fetchBundles = async () => {
  try {
    const response = await fetch(
      `https://up.ft.sa/api/v1/campaign/bundles/${route.params.uuid}`,
      {
        headers: {
          ACCEPT_LANGUAGE: "ar",
        },
      }
    );

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    if (data) {
      console.log("bundles", data.data);
      bundles.value = data.data;
    }
  } catch (err) {
    error.value = "Error fetching products: " + err.message;
    console.error("Error fetching products:", err);
  } finally {
    isLoading.value = false;
  }
};
const fetchGifts = async () => {
  try {
    const response = await fetch(
      `https://up.ft.sa/api/v1/campaign/gifts/${route.params.uuid}`,
      {
        headers: {
          ACCEPT_LANGUAGE: "ar",
        },
      }
    );

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    if (data) {
      console.log("gifts", data.data);
      gifts.value = data.data;
    }
  } catch (err) {
    error.value = "Error fetching products: " + err.message;
    console.error("Error fetching products:", err);
  } finally {
    isLoading.value = false;
  }
};

onMounted(() => {
  fetchCampaign();
  fetchProducts();
  fetchCities();
  fetchBundles();
  fetchGifts();
});
// Watch for changes in form.fullName
watch(
  () => form.fullName,
  (newVal) => {
    if (newVal) {
      errors.fullName = null;
    }
  }
);

// Watch for changes in form.place
watch(
  () => form.place,
  (newVal) => {
    if (newVal) {
      errors.place = null;
    }
  }
);

// Watch for changes in form.selectedCity
watch(
  () => form.selectedCity,
  (newVal) => {
    if (newVal) {
      errors.selectedCity = null;
    }
  }
);

// Watch for changes in form.phoneNumber
watch(
  () => form.phoneNumber,
  (newVal) => {
    if (newVal) {
      errors.phoneNumber = null;
    }
  }
);

const openPopupWindow = (url) => {
  const width = 600;
  const height = 400;
  const left = (window.screen.width - width) / 2;
  const top = (window.screen.height - height) / 2;

  // Open a popup window with custom features
  window.open(
    url,
    "popup",
    `width=${width},height=${height},top=${top},left=${left},resizable=yes,scrollbars=yes`
  );
};

const submitForm = async () => {
  // Reset errors before validation
  resetErrors();

  // Validate each field
  if (!form.fullName) {
    errors.fullName = "الاسم الثلاثى مطلوب";
  }

  if (!form.phoneNumber) {
    errors.phoneNumber = "رقم الجوال مطلوب";
  } else if (!/^\d+$/.test(form.phoneNumber)) {
    errors.phoneNumber = "رقم الجوال غير صحيح";
  }

  if (!form.selectedCity) {
    errors.selectedCity = "المدينة مطلوبة";
  }

  if (!form.place) {
    errors.place = "وصف المكان مطلوب";
  }

  if (otpConfirmed.value === false) {
    errors.otp = "يجب تأكيد رقم الجوال";
  }

  // If there are no errors, proceed
  if (!Object.values(errors).some((error) => error) && otpConfirmed.value === true) {
    const body = {
      phone_number: form.phoneNumber,
      name: form.fullName,
      details: form.place,
      city_id: form.selectedCity,
      product_id: selectedCardData.value.id,
      gift: selectedGift?.value.id || null,
      bundle_ids: selectedBundles.value.map((bundle) => bundle.id),
    };

    const { data, error } = await useFetch(
      `https://up.ft.sa/api/v1/campaign/create-order/${route.params.uuid}`,
      {
        method: "POST",
        body: body,
        headers: {
          ACCEPT_LANGUAGE: "ar",
        },
      }
    );

    if (error.value) {
      let errorsArray = error._value.data.errors;
      if (
        errorsArray.name &&
        Array.isArray(errorsArray.name) &&
        errorsArray.name.length > 0
      ) {
        errors.fullName = errorsArray.name[0];
      }
    } else {
      if (data?.value?.url) {
        window.location.href = data.value.url;
      }
    }
  }
};

const resetErrors = () => {
  errors.fullName = null;
  errors.phoneNumber = null;
  errors.selectedCity = null;
  errors.place = null;
};

const selectCard = (card) => {
  if (selectedCardId.value === card.id) {
    // selectedCardId.value = null;
    // selectedCardData.value = null;
    // currentTab.value = 0;
    // userComments.value = [];
    // usersReviews.value = [];
  } else {
    selectedCardId.value = card.id;
    selectedCardData.value = card;
    currentTab.value = 0;

    userComments.value = card.ratings
      .map((rating) => (!rating.video && !rating.image ? rating : null))
      .filter(Boolean); // Filter out null values

    usersReviews.value = card.ratings
      .map((rating) => (rating.video || rating.image ? rating : null))
      .filter(Boolean); // Filter out null values

    showLoadMoreButton.value =
      userComments.value.length > 2 && displayedComments.value.length === 2;
    if (userComments.value.length > 0) {
      displayedComments.value = userComments.value.slice(0, 2);
    }
  }
};
const loadMoreComments = () => {
  displayedComments.value = userComments.value;
  showLoadMoreButton.value = true;
};
const sortNewestFirst = ref(true); // State to track sorting order
const sortButtonText = ref("الأحدث"); // Initialize with "newest" text

const toggleSort = () => {
  sortNewestFirst.value = !sortNewestFirst.value;

  userComments.value.sort((a, b) => {
    if (!sortNewestFirst.value) {
      return new Date(b.created_at) - new Date(a.created_at); // Newest first
    } else {
      return new Date(a.created_at) - new Date(b.created_at); // Oldest first
    }
  });

  // Update button text based on the sorting order
  sortButtonText.value = sortNewestFirst.value ? "الأحدث" : "الأقدم";

  // After sorting, display the first 2 comments
  displayedComments.value = userComments.value.slice(0, 2);
  showLoadMoreButton.value = false;
};

watch(() => form.phoneNumber, (newValue, oldValue) => {
  otpConfirmed.value = false;
});

watch(selectedBundles, (newValue) => {
  if (newValue.length === 0) {
    selectedGift.value = { price: "0.00" };
  }
});





const handleChange = (value) => {
  console.log(`selected ${value}`);
  console.log("selectedCityData", selectedCityData.value);
  showDelivery.value = true;
  form.value.selectedCity = value;
};

const handleBlur = () => {
  console.log('blur');
};

const handleFocus = () => {
  console.log('focus');
};

const filterOption = (input, option) => {
  return option.label.toLowerCase().includes(input.toLowerCase());
};

const removeBundle = (bundleId) => {
  selectedBundles.value = selectedBundles.value.filter(
    (bundle) => bundle.id !== bundleId
  );
};
const removeGift = () => {
  isDisabled.value = true;
  selectedGift.value = {}
  
};
</script>
