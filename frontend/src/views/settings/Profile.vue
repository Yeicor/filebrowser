<template>
  <div class="row">
    <div class="column">
      <form class="card" @submit="updateSettings">
        <div class="card-title">
          <h2>{{ $t("settings.profileSettings") }}</h2>
        </div>

        <div class="card-content">
          <p>
            <input type="checkbox" v-model="hideDotfiles" />
            {{ $t("settings.hideDotfiles") }}
          </p>
          <p>
            <input type="checkbox" v-model="singleClick" />
            {{ $t("settings.singleClick") }}
          </p>
          <p>
            <input type="checkbox" v-model="dateFormat" />
            {{ $t("settings.setDateFormat") }}
          </p>
          <h3>{{ $t("settings.language") }}</h3>
          <languages
            class="input input--block"
            :locale.sync="locale"
          ></languages>
          <h3>{{ $t("settings.defaultShareDuration") }}</h3>
          <p>{{ $t("settings.defaultShareDurationExplanation") }}</p>
          <div class="input-group input">
            <input
                v-focus
                type="number"
                max="2147483647"
                min="-1"
                v-model.trim="defaultShareDurationTime"
            />
            <select class="right" v-model="defaultShareDurationUnit" :aria-label="$t('time.unit')">
              <option value="seconds">{{ $t("time.seconds") }}</option>
              <option value="minutes">{{ $t("time.minutes") }}</option>
              <option value="hours">{{ $t("time.hours") }}</option>
              <option value="days">{{ $t("time.days") }}</option>
            </select>
          </div>
        </div>

        <div class="card-action">
          <input
            class="button button--flat"
            type="submit"
            :value="$t('buttons.update')"
          />
        </div>
      </form>
    </div>

    <div class="column">
      <form class="card" v-if="!user.lockPassword" @submit="updatePassword">
        <div class="card-title">
          <h2>{{ $t("settings.changePassword") }}</h2>
        </div>

        <div class="card-content">
          <input
            :class="passwordClass"
            type="password"
            :placeholder="$t('settings.newPassword')"
            v-model="password"
            name="password"
          />
          <input
            :class="passwordClass"
            type="password"
            :placeholder="$t('settings.newPasswordConfirm')"
            v-model="passwordConf"
            name="password"
          />
        </div>

        <div class="card-action">
          <input
            class="button button--flat"
            type="submit"
            :value="$t('buttons.update')"
          />
        </div>
      </form>
    </div>
  </div>
</template>

<script>
import { mapState, mapMutations } from "vuex";
import { users as api } from "@/api";
import Languages from "@/components/settings/Languages.vue";
import i18n, { rtlLanguages } from "@/i18n";

export default {
  name: "settings",
  components: {
    Languages,
  },
  data: function () {
    return {
      password: "",
      passwordConf: "",
      hideDotfiles: false,
      singleClick: false,
      dateFormat: false,
      locale: "",
      defaultShareDurationTime: -1,
      defaultShareDurationUnit: "hours",
    };
  },
  computed: {
    ...mapState(["user"]),
    passwordClass() {
      const baseClass = "input input--block";

      if (this.password === "" && this.passwordConf === "") {
        return baseClass;
      }

      if (this.password === this.passwordConf) {
        return `${baseClass} input--green`;
      }

      return `${baseClass} input--red`;
    },
  },
  created() {
    this.setLoading(false);
    this.locale = this.user.locale;
    this.hideDotfiles = this.user.hideDotfiles;
    this.singleClick = this.user.singleClick;
    this.dateFormat = this.user.dateFormat;
    this.defaultShareDurationTime = this.user.defaultShareDurationTime;
    this.defaultShareDurationUnit = this.user.defaultShareDurationUnit;
  },
  methods: {
    ...mapMutations(["updateUser", "setLoading"]),
    async updatePassword(event) {
      event.preventDefault();

      if (this.password !== this.passwordConf || this.password === "") {
        return;
      }

      try {
        const data = { id: this.user.id, password: this.password };
        await api.update(data, ["password"]);
        this.updateUser(data);
        this.$showSuccess(this.$t("settings.passwordUpdated"));
      } catch (e) {
        this.$showError(e);
      }
    },
    async updateSettings(event) {
      event.preventDefault();

      try {
        const data = {
          id: this.user.id,
          locale: this.locale,
          hideDotfiles: this.hideDotfiles,
          singleClick: this.singleClick,
          dateFormat: this.dateFormat,
          defaultShareDurationTime: parseInt(this.defaultShareDurationTime),
          defaultShareDurationUnit: this.defaultShareDurationUnit,
        };
        const shouldReload =
          rtlLanguages.includes(data.locale) !==
          rtlLanguages.includes(i18n.locale);
        await api.update(data, [
          "locale",
          "hideDotfiles",
          "singleClick",
          "dateFormat",
          "defaultShareDurationTime",
          "defaultShareDurationUnit",
        ]);
        this.updateUser(data);
        if (shouldReload) {
          location.reload();
        }
        this.$showSuccess(this.$t("settings.settingsUpdated"));
      } catch (e) {
        this.$showError(e);
      }
    },
  },
};
</script>
