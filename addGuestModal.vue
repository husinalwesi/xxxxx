<template>
  <main v-if="orderData" class="pb-10">
    <SolidBlurBG custom-class="-z-1" :is-bg-image="true" />
    <section class="max-w-1xl mx-auto w-full lg:w-fit add-guests">
      <div class="md:px-10 1xl:px-43 px-4">
        <div class="flex justify-start mb-2 max-w-1xl mx-auto">
          <Breadcrumbs v-if="showBreadCrumbs" :active-index="0" :crumbs="breadcrumbs" />
        </div>
        <div class="page-title mb-8">
          {{ $t('BOOKING_CONFIRMATION.SELECT_GUESTS') }}
        </div>
        <div class="cards-container flex flex-row mobiles:flex-col">
          <div class="w-[648px] mobiles:w-full pb-20 md:pb-0">
            <CardContainer wrapper-classes="px-[15px] pb-[31px] pt-[23px] md:px-[23px] md:py-[31px]">
              <template #card-content>
                <div class="card-title mb-8">
                  <div class="rtl:mb-5">
                    {{
                      $t(`ASSIGN_GUEST.${currentLocation}.TITLE`, {location: $t(`LOCATIONS.${currentLocation}`)})
                    }}
                  </div>
                  <div class="sub-title">
                    {{
                      $t(`ASSIGN_GUEST.${currentLocation}.SUB_TITLE`, {location: $t(`LOCATIONS.${currentLocation}`)})
                    }}
                  </div>
                </div>

                <template v-if="isError">
                  <NoDataFound
                    :no-data-text="$t('SHARED.AN_ERROR_OCCURRED_WHILE_GETTING')"
                    container-classes="my-8"
                    @fetchData="getOrderData()"
                  />
                </template>
                <template v-else>
                  <div v-if="isLoading" class="relative h-20 mt-10">
                    <CircleLoader />
                  </div>
                  <div v-if="!isLoading" class="card-body">
                    <div
                      v-for="(mainTicket, mainIndex) in orderData.collections" :key="mainIndex+refreshId"
                      class="mb-[32px]"
                    >
                      <div
                        class="flex flex-row items-end justify-between pb-2 md:pb-4"
                      >
                        <div class="ltr:font-inter-semi-bold rtl:font-noto-sans-arabic-semi-bold  flex items-center">
                          {{ $getItemTitle(mainTicket) }}
                        </div>
                        <div v-if="mainIndex===0" class="flex flex-col action-details">
                          {{ $t('BOOKING_CONFIRMATION.BOOKING_REFERENCE') }}
                          <div v-if="orderData" class="sub-action pt-1">
                            {{ orderData.reservation_code }}
                          </div>
                        </div>
                      </div>
                      <div class="guest-list flex flex-col gap-y-2">
                        <template v-for="(ticket,ticketIndex) of mainTicket.tickets">
                          <div
                            v-if="ticket.is_addon ? ticket.is_supervision : true"
                            :key="ticket.id"
                            class="guest-card relative"
                            :data-wavier="ticket.need_waiver"
                          >
                            <div class="flex justify-between mobiles:flex-col">
                              <div class="flex flex-col mobiles:mb-2">
                                <div class="name capitalize">
                                  {{
                                    ticket.guest_types[0] ? $t(`COMMON.PEOPLE.${ticket.guest_types[0].title.toUpperCase()}`) : ''
                                  }}
                                </div>
                                <div class="description">
                                  {{ ticketTitle(ticket, mainTicket) }}
                                  &nbsp;{{ getTypeAge(ticket.guest_types, ticket.additional_data) }}
                                </div>
                              </div>
                              <div
                                :class="{ open: selectedMenu === ticket.id }"
                                class="drop-down-btn flex gap-x-2 items-center mobiles:content-start mobiles:justify-end mobiles:flex-row-reverse"
                              >
                                <div class="group inline-block">
                                  <div
                                    :key="ticket.user ? ticket.user.id : -1"
                                    class="selected-guest"
                                    :class="{ 'placeholder': !ticket.user }"
                                    @click.prevent="openDropDown(ticket)"
                                  >
                                    {{
                                      ticket.user
                                        ? ticket.user.name ||
                                          ticket.user.firstName +
                                          " " +
                                          ticket.user.lastName
                                        : $t("ASSIGN_GUEST.CHOOSE_MEMBER")
                                    }}
                                  </div>
                                  <ul
                                    :id="'list-' + ticket.id"
                                    :key="ticket.id"
                                    class="absolute guest-drop-down bg-white pt-2 z-20"
                                    :class="{ hidden: selectedMenu !== ticket.id }"
                                  >
                                    <li
                                      v-for="(user, i) in filterDataByAge(ticket, mainTicket)"
                                      :key="i"
                                      class="list-item"
                                      @click="selectGuest(ticket, user)"
                                    >
                                      <span class="px-4">{{ (user.firstName + ' ' + user.lastName) }}</span>
                                    </li>
                                    <li class="add-new">
                                      <button
                                        type="button"
                                        class="btn-add-card"
                                        @click.prevent="addNewGuest(ticket,mainIndex,ticketIndex)"
                                      >
                                        <span class="icon-wrapper"><Plus fill="#1A1A1A" /></span>
                                        {{ $t('ASSIGN_GUEST.ADD_NEW_GUEST') }}
                                      </button>
                                    </li>
                                  </ul>
                                </div>
                                <ArrowSvg
                                  v-click-outside.special="closeMenu"
                                  :with-wrapper="true"
                                  :wrapper-class="`w-6 h-6 md:w-8 md:h-8 ${
                                    selectedMenu === ticket.id
                                      ? 'bg-dark-gray'
                                      : 'bg-[#ECECEC]'
                                  } open-drop-list`"
                                  :arrow-color="`${
                                    selectedMenu === ticket.id
                                      ? 'white'
                                      : '#0A1432'
                                  }`"
                                  :direction="
                                    selectedMenu === ticket.id ? 'up' : 'down'
                                  "
                                  :width="$globalData.isMobileDevice ? '16' : '21.33'"
                                  :height="$globalData.isMobileDevice ? '16' : '21.33'"
                                  @onArrowClick="openDropDown(ticket)"
                                />
                              </div>
                            </div>
                          </div>
                        </template>
                      </div>
                    </div>
                  </div>
                </template>
              </template>
            </CardContainer>
          </div>
        </div>
      </div>
    </section>
    <BaseModal
      :classes="[`max-w-[40rem] mobiles:max-w-[343px]`]"
      :show-modal="showModal"
      @closeModal="closeModal"
    >
      <h2 slot="header" class="modal-header">
        {{ $t('ASSIGN_GUEST.ADD_NEW_GUEST') }}
      </h2>
      <div slot="content">
        <div v-if="!$isOman()" class="sub-title mb-8">
          {{ $t('BOOKINGFLOW_PAGE.SEND_THE_WAIVER_FORM_TO_YOUR_NEW_GUEST_FORM') }}
        </div>

        <template v-if="showAddLoader">
          <SnowflakeLoader
            loader-wrapper-classes="absolute bg-[#ffffffcb] h-full rounded-lg z-10 inset-0"
            loader-classes="w-10 h-10"
          />
        </template>
        <ValidationObserver ref="add-guest" v-slot="{ invalid }">
          <div class="flex flex-1 flex-wrap gap-y-4 gap-x-8 mb-6">
            <BaseRadio
              v-for="title in titles"
              :key="title.id"
              v-model="formData.title"
              :rules="{ required: true }"
              vid="titles"
              :attr-val="title.value"
              :r-b-i-oname="title.label"
              :r-b-i-olabel="title.label"
              :r-b-i-oid="title.label + title.id"
              label-class="radio-label flex items-center"
            />
          </div>
          <div class="flex flex-row mobiles:flex-col mb-4">
            <div class="w-1/2 ltr:mr-4 rtl:ml-4 mobiles:mb-4 mobiles:w-full">
              <BaseInput
                v-model="formData.firstName"
                i-n-ptype="text"
                i-n-pname="firstName"
                :i-n-pplaceholder="$t('CREDENTIALS_FORM.FIELDS.FIRST_NAME')"
                :classes="['lg:hover:cursor-pointer', 'bg-white']"
                :rules="{required: true, max: 50, alpha_spaces: true}"
              />
            </div>
            <div class="w-1/2 mobiles:w-full">
              <BaseInput
                v-model="formData.lastName"
                i-n-ptype="text"
                i-n-pname="LastName"
                :i-n-pplaceholder="$t('CREDENTIALS_FORM.FIELDS.LAST_NAME')"
                :classes="['lg:hover:cursor-pointer', 'bg-white']"
                :rules="{required: true, max: 50, alpha_spaces: true}"
              />
            </div>
          </div>
          <div class="flex flex-col mb-6">
            <h3 class="mb-2">
              {{ $t('CREDENTIALS_FORM.FIELDS.DATE_OF_BIRTH') }}
            </h3>
            <div class="w-full">
              <BaseDate
                :placeholder="$t('CREDENTIALS_FORM.DD_MM_YYYY')"
                :show-input-title="false"
                :date="formData.date"
                :dynamic-placement="true"
                :extra-rules="{dateRange:dateRange}"
                :is-date-picker-old-days-disabled="false"
                @selectDate="selectDate"
              />
            </div>
          </div>
          <div class="flex flex-col">
            <div>
              <BaseCheckbox
                id="confirm"
                v-model="formData.saveConfirm"
                i-n-pname="confirm-information"
                :label="$t('ASSIGN_GUEST.HEREBY_TEXT')"
                :rules="{ checkboxReq: true }"
              />
            </div>
          </div>
          <div class="border-bottom mt-6" />
          <div class="flex justify-end">
            <BaseButton
              :text="$t('COMMON.SAVE')"
              :b-t-ndisabled="invalid"
              class="btn-yellow text-dark-gray"
              btn-type="button"
              @onBtnClick="submit"
            />
          </div>
        </ValidationObserver>
      </div>
    </BaseModal>
  </main>
</template>

<script>
import BaseModal from "@/components/Shared/BaseUI/BaseModal";
import SolidBlurBG from "@/components/Shared/SolidBlurBG";
import Breadcrumbs from "@/components/Shared/breadcrumbs/breadcrumbs";
import CardContainer from "@/components/BookingFlow/CardContainer";
import BaseInput from "@/components/Shared/BaseUI/FormElements/BaseInput";
import BaseDate from "@/components/Shared/BaseUI/FormElements/BaseDate";
import BaseCheckbox from "@/components/Shared/BaseUI/FormElements/BaseCheckbox/index.vue";
import NoDataFound from "@/components/Accounts/Shared/NoDataFound";
import BaseRadio from "@/components/Shared/BaseUI/FormElements/BaseRadio";
import SnowflakeLoader from "@/components/Shared/Loaders/SnowflakeLoader";
import CircleLoader from "@/components/Shared/Loaders/CircleLoader";

import Plus from "@/components/SVGElements/Plus.vue";
import ArrowSvg from "@/components/SVGElements/arrow-svg.vue";

import {AppConfig} from "@/const/variable";
import {CommonEvent} from "@/services/app-events";

import uniqBy from "lodash/uniqBy";
import clickOutside from '@/directives/click-outside';

export default {  
  name: "AddGuest",
  directives: {
    clickOutside
  },
  components: {
    BaseModal,
    SolidBlurBG,
    BaseDate,
    BaseCheckbox,
    Breadcrumbs,
    CardContainer,
    BaseInput,
    Plus,
    NoDataFound,
    ArrowSvg,
    BaseRadio,
    SnowflakeLoader,
    CircleLoader
  },
  data() {
    return {
      isError: false,
      userDetails: {},
      selectedMenu: -1,
      showModal: false,
      formData: {},
      breadcrumbs: [
        {
          title: this.$t('BOOKING_CONFIRMATION.SELECT_GUESTS'),
          path: "/extra/add-guest",
        },
        {
          title: this.$t('BOOKING_CONFIRMATION.SIGN_WAIVER'),
          path: "/extra/waiver",
        },
        {
          title: this.$t('BOOKING_CONFIRMATION.ADD_GEAR'),
          path: "/extra/gear",
        },
      ],
      titles: [
        {label: this.$t('CREDENTIALS_FORM.TITLES.MR'), id: 1, value: "Mr"},
        {label: this.$t('CREDENTIALS_FORM.TITLES.MRS'), id: 2, value: "Mrs"},
        {label: this.$t('CREDENTIALS_FORM.TITLES.MISS_MS'), id: 3, value: "Miss"}
      ],
      data: [],
      orderData: {},
      tickets: [],
      isLoading: true,
      showAddLoader: false,
      showBreadCrumbs: false,
      tod: new Date(),
      allTickets: [],
      isIncludeMembership: false,
      currentLocation: 'SKI_DUBAI',
      dateRange: {min: 2, max: 9999},
      isRtl: false,
      selectedTicket: null,
      refreshId: 1
    };
  },
  created() {
    this.isRtl = this.$isArabic();
  },
  async mounted() {
    this.currentLocation = this.$getSiteCurrentEnvironment(false, true);
    await this.getOrderData();
    this.$nextTick(() => {
      let wavierItems = document.querySelectorAll('[data-wavier="true"]');
      if (wavierItems && wavierItems.length === 0 && !this.isIncludeMembership) {
        this.breadcrumbs.splice(1, 1);
      }
      this.showBreadCrumbs = true;
    });
  },
  methods: {
    selectGuest(ticket, user) {
      this.closeMenu();
      this.selectedMenu = -1;
      ticket.user = user;
      ticket.user_id = user.id;
      ticket.order_id = this.orderData.id;

      if (ticket.camp_group_id) {
        this.allTickets.forEach((campTicket) => {
          if (campTicket.camp_group_id === ticket.camp_group_id) {
            campTicket.user = user;
            campTicket.user_id = user.id;
            campTicket.order_id = this.orderData.id;
          }
        });
      }

      setTimeout(() => {
        this.checkValidation();
      });
    },
    openDropDown(item) {
      if (this.selectedMenu === item.id) this.selectedMenu = -1;
      else {
        this.selectedMenu = item.id;
        this.$forceUpdate();
      }
    },
    closeMenu() {
      this.selectedMenu = -1;
    },
    getTicketMinMaxAges(ticket) {
      const {ageGroup} = ticket.additional_data ? JSON.parse(ticket.additional_data) : [];

      let min = 0;
      let max = Number.MAX_VALUE;

      if (ageGroup && ageGroup.length && ageGroup[0]) {
        min = ageGroup[0].min || min;
        max = ageGroup[0].max || max;
      } else {
        const type = ticket.guest_types && ticket.guest_types.length ? ticket.guest_types[0].title?.toLowerCase() : "";

        if (type === "adult" || type === "ticket") {
          min = 14;
        } else if (type === "junior") {
          min = 7;
          max = 13;
        } else if (type === "child") {
          min = this.$getSiteCurrentEnvironment(true).includes('oman') ? 2 : 3;
          max = 6;
        }
      }

      return {min, max}
    },
    addNewGuest(ticket, mainTicketIndex, ticketIndex) {
      const {min, max} = this.getTicketMinMaxAges(ticket);

      this.dateRange = {min, max};
      this.selectedTicket = {
        mainTicketIndex,
        ticketIndex,
        ticket
      }
      this.closeMenu();
      this.showModal = true;
    },
    submit() {
      this.$refs["add-guest"].validate().then(async (res) => {
        if (res) {
          await this.createNewSubAccount();
        }
      });
    },
    selectDate(event) {
      this.$nextTick(() => {
        this.formData = Object.assign({}, this.formData, {
          date: this.$moment(event.selectedDate.toString()).format(AppConfig.dateFormat),
        });
      });
    },
    async createNewSubAccount() {
      this.showAddLoader = true;
      const userDetails = {
        email: this.$store.getters["currentUser/userInformation"].email,
        firstName: this.formData.firstName,
        lastName: this.formData.lastName,
        dateOfBirth: this.$dateFormatter(this.formData.date, 'dayMonthYear', false, "YYYY-MM-DD"),
        title: this.formData.title,
      };

      const res = await this.$repositories.createGuest(userDetails);

      if (res) {
        await this.getUsersData(res.data.user);
        this.closeModal();
      } else
        this.$showToast({
          type: "red",
          text: "An error occurred, please try again.",
        });

      this.showAddLoader = false;
    },
    async getUsersData(newUser = null) {
      this.data = await this.$repositories.getUsers();
      if (this.orderData.collections) {
        this.orderData.collections.forEach((item) => {
          item.selectedUser = [];
          if (item.users) {
            let selectedUsers = item.tickets.map((ticket) => ticket.user_id);
            item.users = this.data.filter(
              (user) => !selectedUsers.includes(user.id)
            );
          } else {
            item.users = this.data;
          }
          let isCamps = item.tickets.filter(item => item.camp_group_id)
          if (isCamps.length > 0) {
            this.allTickets = this.allTickets.concat(item.tickets)
            item.tickets = uniqBy(item.tickets, 'camp_group_id')
          }
        });
        if (newUser && this.selectedTicket) {
          this.selectGuest(this.orderData.collections[this.selectedTicket.mainTicketIndex].tickets[this.selectedTicket.ticketIndex], newUser)
          this.refreshId = Date.now();
        }
      }


      this.$forceUpdate();
    },
    async getOrderData() {
      try {
        if (this.isError) this.isError = false;

        this.isLoading = true;

        let order = await this.$repositories.getOrderByCode(this.$route.params.id);

        if (order.isError) {
          this.orderData = {};
          this.isError = true;
        } else {

          if (order.guest_added) {
            this.$router.push(this.localePath(`/booking-confirmed/${order.reservation_code}`));
          }

          let data = []
          order.collections.forEach((item) => {
            if (item.type.toLowerCase() !== "membership" && this.isValidDateEvent(item.event_date)) {
              data.push(item)
            }

            if (item.type.toLowerCase() === 'membership') {
              this.isIncludeMembership = true
            }
          });
          order.collections = data
          this.orderData = order;

          await this.getUsersData();
        }

        this.isLoading = false;
      } catch (err) {
        this.isError = true;
        this.isLoading = false;
        console.log({err});
      }
    },
    async save() {
      CommonEvent.$emit("global-loader", true);

      let tickets = [];
      let dates = []
      for (let i = 0; i < this.orderData.collections.length; i++) {
        let vivaTicket = this.orderData.collections[i];
        dates.push(this.$moment(vivaTicket.event_date).toISOString())
        let ticketsCollections = vivaTicket.tickets.filter((ticket) =>
          ticket.is_addon ? ticket.is_supervision : true
        );
        tickets = tickets.concat(ticketsCollections);
      }

      if (this.allTickets.length > 0) {
        tickets = tickets.filter(item => !item.camp_group_id)
        tickets = tickets.concat(this.allTickets)
      }
      const data = await this.$repositories.assignGuestToTicket(tickets);
      CommonEvent.$emit("global-loader", false);
      if (data) {
        const bookingReference = this.$route.params.id
        this.$repositories.assignGuestEvent(bookingReference, dates)

        let wavierItems = document.querySelectorAll('[data-wavier="true"]');
        if (wavierItems && wavierItems.length === 0 && !this.isIncludeMembership) {
          this.$router.push(this.localePath(`/extra/gear/${bookingReference}?isMembership=${this.isIncludeMembership}&needWaiver=false`));
        } else {
          this.$router.push(this.localePath(`/extra/waiver/${bookingReference}?isMembership=${this.isIncludeMembership}&needWaiver=true&isOnlyMembership=false`));
        }
      }
    },
    filterDataByAge(data, mainTicket) {
      const {min, max} = this.getTicketMinMaxAges(data);

      let users = [];
      let selectedUser = []
      mainTicket.tickets.forEach((item) => {
        if (item.user_id)
          selectedUser.push(item.user_id)
      });
      if (mainTicket.users && mainTicket.users.length) {
        users = mainTicket.users.filter((user) => {
          let userAge = this.getAge(user.dateOfBirth);
          return (
            userAge <= max && userAge >= min && !selectedUser.includes(user.id)
          );
        });
      }

      return users;
    },
    getAge(dateString) {
      var today = new Date();
      var birthDate = new Date(dateString);
      var age = today.getFullYear() - birthDate.getFullYear();
      var m = today.getMonth() - birthDate.getMonth();
      if (m < 0 || (m === 0 && today.getDate() < birthDate.getDate())) {
        age--;
      }
      return age;
    },
    disabledBottomButton() {
      if (document.getElementById("submit-btn"))
        document.getElementById("submit-btn").classList.add("disabled-btn");
    },
    checkValidation() {
      let isValid = true;
      let addGuestLabel = document.querySelectorAll(".placeholder");
      if (addGuestLabel && addGuestLabel.length > 0) {
        isValid = false;
      }
      if (isValid)
        if (document.getElementById("submit-btn"))
          document.getElementById("submit-btn").disabled = false;
    },
    ageGroupTextHandler(age, ticket) {
      let type = ''
      if (ticket[0]) {
        type = ticket[0].title.toLowerCase();
      }
      if (age.min && age.max && type !== 'adult') return this.$t('ASSIGN_GUEST.AGES_BETWEEN', {
        min: age.min,
        max: age.max
      });
      else if (age.min) return `(${this.$t('COMMON.AGES_AND_ABOVE', {age: age.min})})`;

      return "";
    },
    getTypeAge(ticket, additionalData) {
      const {ageGroup} = additionalData ? JSON.parse(additionalData) : [];

      if (ageGroup && ageGroup.length && ageGroup[0]) return this.ageGroupTextHandler(ageGroup[0], ticket);
      else if (ticket[0]) {
        const type = ticket[0].title.toLowerCase();

        if (type === "adult") {
          return `(${this.$t('COMMON.AGES_14_AND_ABOVE')})`;
        } else if (type === "junior") {
          return `(${this.$t('COMMON.AGES_7_13')})`;
        } else if (type === "child") {
          if (this.$isOman()) {
            return `(${this.$t('COMMON.AGES_2_6')})`;
          }
          return `(${this.$t('COMMON.AGES_3_6')})`;
        }
      } else {
        return "";
      }
    },
    ticketTitle(ticket, mainTicket) {

      if (this.isRtl) {
        return ticket.title_ar ? (ticket.title_ar.length > 0 ? ticket.title_ar : mainTicket.title_ar) : mainTicket.title_ar
      }

      return ticket.title_en ? (ticket.title_en.length > 0 ? ticket.title_en : mainTicket.title_en) : mainTicket.title_en
    },
    isValidDateEvent(eventDate) {
      const date = new Date(eventDate);
      if (date.setHours(0, 0, 0, 0) >= this.tod.setHours(0, 0, 0, 0))
        return true;

      return false;
    },
    closeModal() {
      this.showModal = false
      this.formData = {};
      this.selectedTicket = null
    }
  },
};
</script>

<style scoped lang="scss">
@import "AddGuest.scss";
</style>
