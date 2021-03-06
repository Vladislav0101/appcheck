<template>
  <main>
    <Header></Header>
    <section class="picture_actions">
      <component
        :is="SearchUserIs"
        @inputUser="inputUser"
        :idElement="'search'"
        :text="features.search.text"
      ></component>

      <component
        :is="ToSliderButtonIs"
        :idElement="'slider'"
        :text="features.slider.text"
      ></component>
    </section>
    <section class="feed" v-if="arrOfChosenUserPict || arrayOfUrls">
      <PictureBox
        v-for="pictureInfo of arrOfChosenUserPict
          ? arrOfChosenUserPict
          : arrayOfUrls"
        :key="pictureInfo.date"
        :pictureInfo="pictureInfo"
      />
    </section>
    <section v-else>I'm sorry</section>
  </main>
</template>
<script lang="ts">
import Vue from "vue";
import { mapActions, mapGetters } from "vuex";
import throttle from "lodash/throttle";

import { IFeedObject } from "../types/index";

import Header from "@/components/Header/Header.vue";
import PictureBox from "@/components/Main/PictureBox.vue";
import SearchUser from "@/components/Main/SearchUser.vue";
import SearchUserWithHint from "@/components/Main/SearchUserWithHint.vue";
import ToSliderButton from "@/components/Main/ToSliderButton.vue";
import ToSliderButtonWithHint from "@/components/Main/ToSliderButtonWithHint.vue";

interface MainDataProps {
  inputUserValue: string;
  lastCall: Date | null;
  distanceToTheBottom: number;
  throttleScroll: any;
}

export default Vue.extend({
  data(): MainDataProps {
    return {
      inputUserValue: "",
      lastCall: null,
      distanceToTheBottom: 300,
      throttleScroll: throttle(this.handlerScroll, 300)
    };
  },
  methods: {
    ...mapActions(["getPictures"]),

    inputUser(value: string): void {
      this.inputUserValue = value;
    },

    handlerScroll() {
      const totalHeight = document.documentElement.scrollHeight;
      const scrollTop = window.pageYOffset;
      const displayHeight = document.documentElement.clientHeight;

      if (
        totalHeight - scrollTop - displayHeight <= this.distanceToTheBottom &&
        this.isInfiniteScrollEnabled
      ) {
        this.getPictures();
      }
    },

    subscribeToScroll(): void {
      window.addEventListener("scroll", this.throttleScroll);
    }
  },

  beforeDestroy(): void {
    window.removeEventListener("scroll", this.throttleScroll);
  },

  mounted(): void {
    if (!this.arrayOfUrls.length) {
      this.getPictures();
    }
    this.subscribeToScroll();
  },

  computed: {
    ...mapGetters([
      "arrayOfUrls",
      "isLearningPathActive",
      "features",
      "version",
      "isInfiniteScrollEnabled"
    ]),

    arrOfChosenUserPict(): string | Array<IFeedObject> {
      if (this.inputUserValue) {
        return this.arrayOfUrls.filter((item: IFeedObject) => {
          return item.email === this.inputUserValue;
        });
      } else {
        return "";
      }
    },

    SearchUserIs(): Function {
      return this.isLearningPathActive && this.version === "1.0"
        ? SearchUserWithHint
        : SearchUser;
    },

    ToSliderButtonIs(): Function {
      return this.isLearningPathActive && this.version === "1.0"
        ? ToSliderButtonWithHint
        : ToSliderButton;
    }
  },

  components: {
    Header,
    PictureBox,
    SearchUser,
    SearchUserWithHint,
    ToSliderButton,
    ToSliderButtonWithHint
  }
});
</script>
<style scoped>
.picture_actions {
  display: flex;
  margin-top: 10px;
  align-items: center;
  justify-content: center;
  gap: 15px;
}
.feed {
  display: grid;
  justify-content: center;
}
</style>
