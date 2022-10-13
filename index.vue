<template>
  <main class="c-qrcode" v-if="content">
    <section class="c-hero is-relative">
      <div class="container">
        <div class="c-hero-content">
          <div class="c-hero-content-information p-x-16-touch">
            <h1 class="c-hero-content-information-title has-text-white" v-html="content.body.hero.title" />
            <p class="c-hero-content-information-subtitle has-text-cielo-600" v-html="content.body.hero.subtitle" />
          </div>
          <ResponsiveImage
            :lazyload="false"
            :sources="content.body.hero.image.src"
            :alt="content.body.hero.image.alt"
            class="c-hero-content-image"
          />
        </div>
      </div>
    </section>

    <section class="c-news is-relative">
      <div class="container p-x-16-touch">
        <p class="c-news-description has-text-centered is-size-11-touch lh-12-touch is-size-6 lh-6 p-t-40 has-text-cloudy-400 has-text-weight-medium" v-html="content.body.news.description" />
        <ul class="is-flex c-news-list p-t-40-touch p-t-72">
          <li 
            v-for="(item, i) in content.body.news.list"
            :key="i"
            class="c-news-list-items is-flex m-b-16-touch"
          >
            <div class="c-news-list-items-image m-r-12-touch">
              <img class="c-news-list-items-icon" :src="item.icon.src" :alt="item.icon.alt">
            </div>
            <p v-html="item.information" class="c-news-list-items-information has-text-centered is-size-12-touch lh-12-touch is-size-9 lh-9 has-text-weight-bold has-text-cloudy-400" />
          </li>
        </ul>
      </div>
    </section>

    <section class="c-payment is-relative is-flex p-t-32-touch p-b-48-touch p-t-48 p-b-96">
      <h2 class="c-payment-title is-size-9-touch lh-9-touch is-size-6 lh-6 has-text-cielo-300 has-text-weight-medium has-text-centered p-b-32-touch p-b-20 p-t-20-touch p-x-16-touch" v-html="content.body.payment.title" />
      <div class="options m-t-12 m-t-12-touch p-l-16-touch p-l-32 is-flex">
        <CardSlider 
          ref="slider"
          :queries="slider_config"
          class="c-payment-slider"
        >
          <BRadio 
            v-for="(flow, index) in content.body.payment.flow"
            :key="index"
            name="machine-options"
            :native-value="index"
            v-bind="flow.option.trackData"
            @input="$emit('input', flow.option.id)"
            class="machine-options m-x-12 m-y-16-touch has-background-white"
            :class="{'specific-style': index == 1}"
            v-model="optionMain"
            @click.native="updateFlow(index)"
          >
            <ResponsiveImage :sources="flow.option.image.src" :alt="flow.option.image.alt" class="machine-image" />
            <div class="label m-l-16-touch p-t-0-touch">
              <p class="has-text-cielo-500 is-size-9 lh-9 is-size-11-mobile lh-8 lh-12-touch" v-html="flow.option.title" /> 
              <p class="is-size-10 is-size-12-mobile lh-10 lh-12-touch has-text-weight-normal has-text-cloudy-400" v-html="flow.option.description" /> 
            </div>
          </BRadio>
        </CardSlider>
      </div>
      <div class="c-payment-navigate has-text-cielo-400 has-text-weight-medium">
        Navegue
      </div>
      <div class="c-payment-flow">
        <div class="c-payment-flow-next-prev" v-if="hasNextPrev">
          <button
            @click="gotNextPrev"
            class="c-payment-flow-next-prev-button"
            :class="{ 'is-active' : indexDelivery == 0 }"
            v-bind="content.body.payment.flow[currentIndex].steps[indexDelivery].delivery.trackData"
          >
            Com Frete
          </button>
          <button
            @click="gotNextPrev"
            class="c-payment-flow-next-prev-button"
            :class="{ 'is-active' : indexDelivery == 1 }"
            v-bind="content.body.payment.flow[currentIndex].steps[indexDelivery].delivery.trackData"
          >
            Sem Frete
          </button>
        </div>
        <FlowContainer 
          :content="getData()" 
          :hideArrows="true" 
          v-model="firstFlow"
          class="p-t-0-touch p-t-32"
          :class="`c-payment-flow-${getData().option.id}`"
          :value="3"
          @click="firstFlow = 2"
          :key="forceUpdate"
        />
      </div>
      <ResponsiveImage
        :lazyload="false"
        :sources="content.body.payment.separator.src"
        :alt="content.body.payment.separator.alt"
        class="c-payment-separator"
      />
    </section>

    <section class="c-challenge is-relative">
      <div class="container is-flex p-x-16-touch">
        <div class="c-challenge-informations">
          <h2 class="c-challenge-informations-title has-text-cielo-300 is-size-10-touch lh-9-touch is-size-6 lh-6 has-text-weight-medium" v-html="content.body.challenge.title" />
          <p class="c-challenge-informations-description has-text-cielo-600 has-text-weight-medium p-t-20" v-html="content.body.challenge.description" />
        </div>
        <ResponsiveImage
          :lazyload="false"
          :sources="content.body.challenge.image.src"
          :alt="content.body.challenge.image.alt"
          class="c-challenge-image p-t-16-touch"
        />
      </div>
    </section>

    <section class="c-benefits is-relative">
      <div class="container is-flex p-x-16-touch">
        <ResponsiveImage
          :lazyload="false"
          :sources="content.body.benefits.image.src"
          :alt="content.body.benefits.image.alt"
          class="c-benefits-image p-t-40-touch"
        />
        <div class="c-benefits-resources p-l-0-touch p-l-40">
          <h2 class="p-t-40-touch p-t-72 p-b-32-touch p-b-24 is-size-9-touch lh-9-touch is-size-6 lh-6 has-text-cielo-300 has-text-weight-medium" v-html="content.body.benefits.title" />
          
          <ul class="c-benefits-resources-list is-flex">
            <li 
              v-for="(item, i) in content.body.benefits.list"
              :key="i"
              class="c-benefits-resources-list-item p-x-8 p-t-24 p-b-16 is-relative"
            >
              <img class="c-benefits-resources-list-items-icon is-block" :src="item.icon.src" :alt="item.icon.alt">
              <p class="c-benefits-resources-list-items-description is-size-12-touch lh-15-touch is-size-10 lh-10 has-text-centered has-text-cloudy-400 has-text-weight-medium" v-html="item.description" />
              <AppLink class="c-benefits-resources-list-items-button button has-background-white has-text-cloudy-400 is-size-12-touch lh-15-touch is-size-10 lh-10"
                       v-if="item.link"
                       :to="item.link.href"
                       :target="item.link.target"
                       v-bind="item.link.trackData"
                       v-html="item.link.label" 
              />
            </li>
          </ul>
        </div>
      </div>
      <ResponsiveImage
        :lazyload="false"
        :sources="content.body.benefits.separator.src"
        :alt="content.body.benefits.separator.alt"
        class="c-benefits-separator"
      />
    </section>

    <section class="c-flags is-relative m-b-32-touch m-b-80 p-t-32">
      <div class="container">
        <h2 class="is-size-9-touch lh-9-touch is-size-6 lh-6 has-text-cielo-300 has-text-weight-medium has-text-centered p-b-20" v-html="content.body.flags.title" />
        <CardSlider 
          ref="slider"
          :queries="slider_config"
          class="c-flags-slider"
        >
          <li 
            v-for="(item, i) in content.body.flags.list" :key="i"
            class="c-flags-slider-card is-flex m-x-12"
          >
            <div
              v-for="(icon, j) in item.icon" :key="j"
              class="c-flags-slider-card-icon is-flex m-y-12 p-8"
            >
              <img class="c-flags-slider-card-icon-image" :src="icon.src" :alt="icon.alt" :title="icon.alt">
            </div>
          </li>
        </CardSlider>
      </div>
    </section>

    <section class="c-contact is-relative">
      <div class="container">
        <div class="items">
          <div class="item">
            <h4>
              <span class="icon-phone" />
              Central de relacionamento:
            </h4>
            <div class="phones">
              <div class="phone">
                <p>
                  <span class="icon-number" />
                  <span class="phone-text">4002 5472</span>
                </p>
                <p class="days-text">
                  (Capitais e regiões metropolitanas)
                </p>
              </div>
              <div class="phone">
                <p>
                  <span class="icon-global" />
                  <span class="phone-text">0800 570 5472</span>
                </p>
                <p class="days-text">
                  (Exceto capitais e ligações de celulares)
                </p>
              </div>
            </div>
            <p class="detail-text">
              Todos os dias, <strong>24 horas por dia</strong>
            </p>
          </div>
          <div class="item">
            <h4>
              <span class="icon-suporte" />
              Suporte Técnico:
            </h4>
            <div class="phones">
              <div class="phone">
                <p>
                  <span class="icon-number" />
                  <span class="phone-text">4002 9111</span>
                </p>
                <p class="days-text">
                  (Capitais e regiões metropolitanas)
                </p>
              </div>
              <div class="phone">
                <p>
                  <span class="icon-global" />
                  <span class="phone-text">0800 570 0111</span>
                </p>
                <p class="days-text">
                  (Exceto capitais e ligações de celulares)
                </p>
              </div>
            </div>
            <p class="detail-text">
              Todos os dias, <strong>24 horas por dia</strong>
            </p>
          </div>
          <div class="item">
            <h4>
              <span class="icon-ouvidoria" />
              Ouvidoria:
            </h4>
            <div class="phones">
              <div class="phone">
                <p>
                  <span class="phone-text">0800 570 2288</span>
                </p>
                <p class="detail-text">
                  Se não ficar satisfeito com a solução apresentada, contate Ouvidoria com protocolo em mãos. De segunda a sexta, das 8h às 18h.
                </p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>
  </main>
</template>

<script>
import ResponsiveImage from '~/components/shared/ResponsiveImage'
import CardSlider from "~/components/shared/CardSlider";
import { jsonLoader } from '~/utils/jsonLoader.js'
import FlowContainer from '~/components/shared/FlowContainer'
import { BRadio } from 'buefy/dist/components/radio'

const route = '/qrcode'

export default {

  head () {
    return this.$createHeadFunction(
      this.content.meta,
      this.$store.state.headerFooterContent.header.favicon,
      this.$route.path,
      JSON.stringify(this.content.dataLayer)
    );
  },

  components: {
    ResponsiveImage,
    CardSlider,
    FlowContainer,
    BRadio
  },

  async asyncData({}) {
    return jsonLoader(route) 
  },
  
  data() {
    return{
      slider_config: [
        {
          max_width: 9999,
          config: {
            pageDots: false,
            imagesLoaded: false,
            cellAlign: "center",
            contain: true,
            wrapAround: false,
            draggable: true,
            freeScroll: false,
            prevNextButtons: false,
          },
        },
      ],
      firstFlow: 0,
      optionMain: 0,
      indexDelivery: 0,
      currentIndex: 0,
      forceUpdate: 0,
      hasNextPrev: false
    }
  },

  methods: {
    updateFlow: function(index) {
      this.forceUpdate += 1
      this.currentIndex = index
    },

    getData() {
      if (this.content.body.payment.flow[this.currentIndex].steps[this.indexDelivery].delivery) {
        this.hasNextPrev = true
        return this.content.body.payment.flow[this.currentIndex].steps[this.indexDelivery].delivery
      } else {
        this.hasNextPrev = false
        return this.content.body.payment.flow[this.currentIndex]
      }
    },

    gotNextPrev() {
      if(this.indexDelivery == 0) {
        this.indexDelivery++
      } else if(this.indexDelivery > 0) {
        this.indexDelivery--
      }
      this.forceUpdate += 1
    },

    forceRebuild() {
      this.forceUpdate += 1
    }
  },
}
</script>

<style lang="scss" scoped>
@import "~assets/css/abstract/index";

.c-qrcode {
  .c-hero {
    height: 499px;
    z-index: 9;

    @include until ($desktop) {
      width: 361px;
      height: 368px;
      margin: auto;
    }
    
    .c-hero-content {
      max-width: 599px;
      margin: 0 0 0 auto;

      @include until ($fullhd) {
        max-width: 512px;
      }

      @include until ($widescreen) {
        max-width: 434px;
      }

      .c-hero-content-information {
        .c-hero-content-information-title,
        .c-hero-content-information-subtitle {
          position: relative;
          z-index: 1;

          @include until ($desktop) {
            width: 300px;
            /deep/br {
              display: none;
            }
          }
        }

        .c-hero-content-information-title {
          padding: 69px 0 40px;
          font-size: 32px;
          line-height: 44px;

          @include until ($desktop) {
            padding: 21px 0 8px;
            font-size: 16px !important;
            line-height: 22px !important;
          }

          @include until ($fullhd) {
            font-size: 28px;
            line-height: 28px;
          }

          @include until ($widescreen) {
            font-size: 22px;
            line-height: 22px;
          }
        }

        .c-hero-content-information-subtitle {
          font-size: 24px;
          line-height: 32px;

          @include until ($desktop) {
            font-size: 12px !important;
            line-height: 20px !important;
          }

          @include until ($fullhd) {
            font-size: 22px;
            line-height: 30px;
          }

          @include until ($widescreen) {
            font-size: 18px;
            line-height: 24px;
          }
        }
      }
      
      .c-hero-content-image {
        width: 100%;
        position: absolute;
        top: 0;
        left: 0;
      }
    }
  }

  .c-news {
    z-index: 9;

    .container {
      max-width: 1008px;

      @include until ($desktop) {
        max-width: 360px;
      }
    }

    .c-news-description {
      @include until ($desktop) {
        /deep/br {
          display: none;
        }
      }
    }

    .c-news-list {
      width: 805px;
      margin: auto;
      justify-content: space-between;

      @include until ($desktop) {
        width: auto;
        flex-direction: column;
      }
      
      .c-news-list-items {
        --icon_size: 104px;
        --circle_size: 183px;

        @include until ($desktop) {
          --icon_size: 27px;
          --circle_size: 48px;
        }

        flex-direction: column;
        width: var(--circle_size);

        @include until ($desktop) {
          width: 100%;
          flex-direction: row;
        }

        .c-news-list-items-image {
          @include until ($desktop) {
            width: var(--circle_size);
            min-height: var(--circle_size);
          }

          &::before {
            content: "";
            width: var(--circle_size);
            height: var(--circle_size);
            background: $cielo_300;
            position: absolute;
            z-index: 1;
            border-radius: 100%;
          }

          .c-news-list-items-icon {
            width: var(--icon_size);
            height: var(--icon_size);
            z-index: 2;
            position: relative;
            margin-left: calc((#{var(--circle_size)} - #{var(--icon_size)})/2);
            margin-top: calc((#{var(--circle_size)} - #{var(--icon_size)})/2);
          }
        }

        .c-news-list-items-information {
          width: 100%;
          margin-top: calc((#{var(--circle_size)} - #{var(--icon_size)})/2 + 20px);
          
          @include until ($desktop) {
            margin: auto;
          }
        }
      }
    }
  }

  .c-payment {
    flex-direction: column;

    &::after {
      content: "";
      height: 500px;
      width: 100%;
      background: $cloudy-100;
      display: block;
      position: absolute;
      bottom: -45px;
      z-index: 0;

      @include until ($desktop) {
        bottom: -100px;
      }
    }

    .c-payment-separator {
      position: absolute;
      bottom: 200px;
      z-index: 1;
      width: 100%;

      @include until ($desktop) {
        bottom: 280px;
      }

      /deep/img {
        width: 100%;
      }
    }

    .container {
      max-width: 1008px;
      z-index: 9;

      @include until ($desktop) {
        max-width: 360px;
      }
    }

    .c-payment-title {
      order: 1;

      @include until ($desktop) {
        order: 3;
      }
    }

    .c-payment-navigate {
      display: flex;
      align-items: center;
      justify-content: center;
      width: 100%;
      order: 2;
      position: relative;
      z-index: 9;

      @include until ($desktop) {
        order: 1;
      }

      &::before, &::after {
        content: "";
        width: 0; 
        height: 0; 
        border-left: 6px solid transparent;
        border-right: 6px solid transparent;
        border-top: 5px solid #017CEB;
      }

      &::before{
        margin-right: 12px;
        transform: rotate(90deg);
      }

      &::after{
        margin-left: 12px;
        transform: rotate(270deg);
      }
    }

    .c-payment-flow {
      z-index: 9;
      order: 2;
      position: relative;

      @include until ($desktop) {
        order: 4;
      }

      .c-payment-flow-next-prev {
        max-width: 1086px;
        margin: auto;
        display: flex;
        justify-content: center;
        position: absolute;
        top: 62px;
        z-index: 9;
        left: calc(50% - 536px/2);

        @include until ($desktop) {
          bottom: -76px;
          position: absolute;
          top: 40px;
          left: calc(50% - 136px);
        }

        .c-payment-flow-next-prev-button {
          background: $white;
          border: 1px solid $cloudy_200;
          box-sizing: border-box;
          width: 268px;
          height: 40px;
          font-weight: 500;
          font-size: 16px;
          line-height: 16px;
          color: $cloudy_400;
          cursor: pointer;

          @include until ($desktop) {
            width: 136px;
            height: 32px;
            font-size: 14px;
            line-height: 14px;
          }

          &:first-child {
            border-radius: 8px 0px 0px 8px;
            border-right: 0px;
          }

          &:last-child {
            border-radius: 0px 8px 8px 0px;
          }

          &.is-active {
            background: $cielo_300;
            color: $white;
          }
        }
      }

      // images alignment on flow slider
      .c-payment-flow-lio,
      .c-payment-flow-zip,
      .c-payment-flow-flash,
      .c-payment-flow-ecommerce,
      .c-payment-flow-superlink {
        /deep/picture {
          position: relative;
          
          @include from ($desktop) {
            top: 84px;
          }
        }
      }

      .c-payment-flow-lio {
        /deep/picture {
          @include from ($desktop) {
            top: 84px;
          }
        }
      }

      .c-payment-flow-zip {
        /deep/picture {
          @include from ($desktop) {
            top: 70px;
          }
        }
      }

      .c-payment-flow-flash {
        /deep/picture {
          @include from ($desktop) {
            top: 70px;
          }
        }
      }

      .c-payment-flow-ecommerce {
        /deep/picture {
          @include from ($desktop) {
            top: 22px;
          }
        }
      }

      .c-payment-flow-superlink {
        /deep/picture {
          @include from ($desktop) {
            top: 20px;
          }
        }
      }
    }

    /deep/.flow {
      order: 3;
      z-index: 9;

      .platform-flow {
        position: absolute;
        top: 0;
        box-shadow: 0px 1px 4px rgba(35, 31, 32, 0.32);

        @include until ($desktop) {
          min-height: 600px !important;
          padding-top: 100px !important;
          padding-bottom: 36px !important;
        }

        .responsive-image {
          z-index: 10;
        }

        .description-container {
          justify-content: space-around;
          max-width: 420px;

          span {
            font-weight: 500;
          }
        }
      }

      .description-content {
        padding-bottom: 20px;
      }

      .description-content {
        span {
          &:first-child {
            color: $cielo_300 !important;
          }
        }
      }

      .dot.is-active {
        background: $cielo_300 !important;
      }
    }

    .c-payment-slider {
      width: 100%;

      /deep/.flickity-viewport {
        @include from ($desktop) {
          height: 167px !important;
          padding: 30px 0;
        }

        .b-radio {
          @include until ($desktop) {
            &:first-child {
              margin-left: 20px !important;
              left: 1% !important;
            }
          }
        }
      }
    }

    .options{
      min-height: 117px !important;
      justify-content: center;
      order: 2;
      z-index: 9;

      @include until ($desktop) {
        order: 2;
      }

      @include mobile {
        flex-direction: column;
        align-items: center;

        @include from ($desktop) {
          height: 380px;
        }
      }

      .machine-options{
        width: 293px;
        height: 117px !important;
        border: 1px solid #C5CED7;
        border-radius: 9px;
        display: flex;
        align-items: flex-end;

        &:nth-child(1) {
          .machine-image {
            width: 45%;
            margin-bottom: 0;

            @include until ($desktop) {
              height: 117px;
            }

            /deep/img {
              margin-top: -28px;

              @include until ($desktop) {
                margin-top: 0;
              }
            }
          }
        }

        &:nth-child(2) {
          .machine-image {
            @include until ($desktop) {
              height: 117px;
            }

            /deep/img {
              margin-top: -22px;
            }
          }
        }

        &:nth-child(3) {
          .machine-image {
            @include until ($desktop) {
              height: 117px;
            }

            /deep/img {
              margin-top: -14px;
            }
          }
        }

        &:nth-child(4),
        &:nth-child(5) {
          .machine-image {
            @include until ($desktop) {
              height: 117px;
            }

            /deep/img {
              margin-top: 10px;
            }
          }
        }

        @include mobile {
          width: 293px;
          min-height: 117px !important;
        }

        /deep/.control-label{
          display: flex;

          @include desktop{
            width: calc(100% - 24px);
            margin: 0 auto;
            padding-left: 0px;
            height: 117px;
          }


          @include mobile {
            width: 100%;
          }
        }

        /deep/.check{
          position: absolute;
          border: 1px solid #C5CED7;
          top: 8px;
          right: 8px;
          height: 24px;
          width: 24px;

          @include touch {
            height: 18px;
            width: 18px;
          }
        }

        .machine-image{
          margin-bottom: -15px;
          min-width: 108px;
          text-align: center;

          @include mobile {
            width: 40%;
            max-width: 150px;

            /deep/img{
              max-height: 120px;
              max-width: 120px;
            }
            
          }
        }

        .label{
          display: flex;
          flex-direction: column;
          // height: 95px;
          // margin-top: auto;
          // margin-bottom: 0px;
          margin: auto 0;
          max-width: 190px;

          @media(max-width: 1215px)and(min-width: 769px){
            margin-left: 16px !important;
            padding-top: 8px !important;
          }

          @include mobile {
            max-width: 60%;
            width: 145px;
          }

          /deep/strong{
            font-weight: 700;
          }
        }
      }
    }

    .button{
      max-width: 497px;

      @include touch {
        max-width: 270px;
      }
    }

    .disabled{
      pointer-events: none;
      background-color: #C5CED7 !important;
    }
  }

  .c-challenge {
    --image_margin_top: 40px;

    background: $cloudy-100;
    z-index: 9;

    @include from ($desktop) {
      min-height: 459px;
    }

    .container {
      max-width: 1008px;
      height: 100%;
      margin-top: var(--image_margin_top);

      @include until ($desktop) {
        max-width: 360px;
      }

      @include from ($desktop) {
        min-height: 459px;
      }

      @include until ($desktop) {
        margin-top: auto;
        flex-direction: column;
      }
    }

    .c-challenge-informations {
      @include from ($desktop) {
        margin: auto 0;
      }

      .c-challenge-informations-title {
        @include until ($desktop) {
          /deep/br {
            display: none;
          }
        }
      }

      .c-challenge-informations-description {
        font-size: 18px;
        line-height: 24px;

        @include until ($desktop) {
          /deep/br {
            display: none;
          }
        }

        @include until ($desktop) {
          font-size: 14px;
          line-height: 20px;
        }

        /deep/strong {
          font-size: 20px;
          line-height: 26px;

          @include until ($desktop) {
            font-size: 16px;
            line-height: 22px;
          }
        }

        /deep/span {
          font-size: 14px;
          line-height: 20px;

          @include until ($desktop) {
            font-size: 10px;
            line-height: 20px;
          }
        }
      }
    }

    .c-challenge-image {
      margin: 0 auto;

      @include from ($desktop) {
        position: absolute;
        right: -113px;
        top: calc(-1 * var(--image_margin_top));
      }
    }
  }

  .c-benefits {
    background: $cloudy-100;
    z-index: 9;

    .c-benefits-separator {
      position: absolute;
      bottom: -50px;
      width: 100%;

      @include until ($desktop) {
        bottom: -10px;
      }

      /deep/img {
        width: 100%;
      }
    }

    .container {
      max-width: 1008px;
      z-index: 9;

      @include until ($desktop) {
        max-width: 360px;
      }

      @include until ($desktop) {
        flex-direction: column-reverse;
      }
    }

    .c-benefits-image {
      @include until ($desktop) {
        margin: auto;
      }
    }

    .c-benefits-resources {
      .c-benefits-resources-list {
        @include until ($desktop) {
          justify-content: center;
        }

        .c-benefits-resources-list-item {
          width: 142px;
          background: $white;
          border: 1px solid $cloudy-200;
          border-radius: 12px;

          @include until ($desktop) {
            width: 102px;
          }

          &:not(:last-child) {
            margin-right: 54px;

            @include until ($desktop) {
              margin-right: 10px;
            }
          }

          .c-benefits-resources-list-items-icon {
            margin: 0 auto 16px;
            width: 64px;
            height: 64px;

            @include until ($desktop) {
              width: 48px;
              height: 48px;
            }
          }

          .c-benefits-resources-list-items-description {
            /deep/span {
              display: block;
              font-size: 10px;
              line-height: 10px;
              padding-top: 16px;

              @include until ($desktop) {
                font-size: 8px;
                line-height: 10px;
              }
            }
          }

          .c-benefits-resources-list-items-button {
            position: absolute;
            bottom: -42px;
            width: calc(100% - 8px);
            left: 4px;
            border-color: $cielo_300;

            @include until ($desktop) {
              bottom: -26px;
            }
          }
        }
      }
    }
  }

  .c-flags {
    z-index: 9;

    .container {
      max-width: 1008px;

      @include until ($desktop) {
        max-width: 360px;
      }
    }

    .c-flags-slider {
      .c-flags-slider-card {
        flex-direction: column;

        .c-flags-slider-card-icon {
          width: 95px;
          height: 95px;
          border: 1px solid $cloudy-200;
          box-sizing: border-box;
          border-radius: 9.5px;

          @include until ($desktop) {
            width: 80px;
            height: 80px;
          }
        }

        .c-flags-slider-card-icon-image {
          margin: auto;
        }
      }
    }
  }
  
  .c-contact {
    z-index: 9;
    background-color: $cielo-300;
    color: white;
    padding: 2rem 0;

    .items {
      display: flex;
      flex-direction: column;
      margin: 0 auto;
      width: 100%;
      @include from ($desktop) {
        max-width: 900px;
        flex-direction: row;
      }

      .item {
        width: 100%;
        box-sizing: border-box;
        max-width: 278px;
        margin: 0 auto;
        position: relative;
        @include from ($desktop) {
          max-width: 325px;
          padding: 0 1.5rem;
          margin: 0;
        }

        &:nth-child(2) {
          padding: 28px 0;
          margin: 28px auto;
          position: relative;

          @include from ($desktop) {
            padding: 0 1.5rem;
            margin: 0;
          }

          &:before, &:after {
            content: "";
            width: 68px;
            height: 1px;
            border-top: 1px solid #80d7f7;
            display: block;
            position: absolute;
            left: 50%;
            margin-left: -34px;
            top: 0;

            @include from ($desktop) {
              height: 68px;
              width: 1px;
              border-left: 1px solid $cielo-600;
              left: 0;
              top: 50%;
              margin-left: 0;
              margin-top: -34px;
            }
          }

          &:after {
            top: initial;
            bottom: 0;

            @include from ($desktop) {
              right: 0;
              top: 50%;
              bottom: initial;
              left: initial;
            }
          }
        }

        &:last-child {
          width: 185px;
          @include from ($desktop) {
            width: 260px;
          }
        }

        h4 {
          font-weight: 700;
          font-size: 14px;
          line-height: 16px;
        }

        .phones {
          margin-bottom: 0.65rem;
          margin-top: 1rem;
          display: flex;

        }

        .icon-number,
        .icon-global,
        .icon-phone,
        .icon-suporte,
        .icon-ouvidoria {
          width: 20px;
          height: 20px;
          display: inline-block;
          vertical-align: sub;
        }

        .icon-number {
          background: url('/assets_cielo/icons/LightBlue/Location&Transport/map-pin.svg') no-repeat 0 0;
        }

        .icon-global {
          background: url('/assets_cielo/icons/LightBlue/Location&Transport/globe.svg') no-repeat 0 0;
        }

        .icon-phone {
          background: url('/assets_cielo/icons/LightBlue/Communication/phone.svg') no-repeat 0 0;
        }

        .icon-suporte {
          background: url('/assets_cielo/icons/LightBlue/Others/settings.svg') no-repeat 0 0;
        }

        .icon-ouvidoria {
          background: url('/assets_cielo/icons/LightBlue/Communication/chat-multiple.svg') no-repeat 0 0;
        }

        .phone-text {
          font-weight: 700;
          font-size: 16px;
          line-height: 18px;
        }

        .detail-text,
        .days-text {
          font-size: 10px;
          line-height: 11px;
        }
      }
    }
  }
}

</style>
