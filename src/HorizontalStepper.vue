<template>
    <div class="stepper-box box">
        <div class="top">
            <div class="divider-line" :style="{width: `${(100/(steps.length) * (steps.length - 1)) - 10}%`}"></div>
            <div class="steps-wrapper">
                <template v-if="topButtons">
                    <div v-if="currentStep.index > 0" class="stepper-button-top previous" @click="backStep()">
                        <span class="icon">
                            <i class="fa fa-arrow-left"></i>
                        </span>
                    </div>
                </template>
                <template v-for="(step, index) in steps">
                    <div :class="['step', isStepActive(index, step)]" :key="index"
                         :style="{width: `${100 / steps.length}%`}">
                        <div class="circle">
                            <span class="icon is-large">
                                <i :class="'fa fa-2x '+(step.completed ? 'fa-check' : step.icon)"></i>
                            </span>
                        </div>
                        <div class="step-title">
                            <h4>{{step.title}}</h4>
                            <h5 class="step-subtitle">{{step.subtitle}}</h5>
                        </div>
                    </div>
                </template>
                <div v-if="topButtons" :class="['stepper-button-top next', !canContinue ? 'deactivated' : '']"
                     @click="nextStep()">
                    <span class="icon">
                        <i class="fa fa-arrow-right"></i>
                    </span>
                </div>
            </div>
        </div>
        <div class="content">
            <transition :enter-active-class="enterAnimation" :leave-active-class="leaveAnimation" mode="out-in">
                <!--If keep alive-->
                <keep-alive v-if="keepAlive">
                    <component :is="steps[currentStep.index].component" :clickedNext="nextButton[currentStep.name]"
                               @can-continue="proceed" @change-next="changeNextBtnValue"
                               @data-update="onDataUpdate"
                               :current-step="currentStep"></component>
                </keep-alive>
                <!--If not show component and destroy it in each step change-->
                <component v-else :is="steps[currentStep.index].component" :clickedNext="nextButton[currentStep.name]"
                           @can-continue="proceed" @change-next="changeNextBtnValue"
                           @data-update="onDataUpdate"
                           :current-step="currentStep"></component>
            </transition>
        </div>
        <div :class="['bottom', (currentStep.index > 0) ? '' : 'only-next']">
            <div v-if="currentStep.index > 0" class="button previous" @click="backStep()">
                <span class="icon">
                    <i class="fa fa-arrow-left"></i>
                </span>
                <span>{{ 'back' | translate(locale) }}</span>
            </div>
            <div :class="['button next', !canContinue ? 'deactivated' : '']" @click="nextStep()">
                <span>{{ (finalStep) ? 'finish' : 'next' | translate(locale) }}</span>
                <span class="icon">
                    <i class="fa fa-arrow-right"></i>
                </span>
            </div>
        </div>
    </div>
</template>

<script>
import translations from './Translations.js'

export default {
  filters: {
    translate: function (value, locale) {
      return translations[locale][value]
    }
  },

  props: {
    locale: {
      type: String,
      default: 'en'
    },
    topButtons: {
      type: Boolean,
      default: false
    },
    steps: {
      type: Array,
      default: function () {
        return [
          {
            icon: 'fa-envelope',
            name: 'first',
            title: 'Sample title 1',
            subtitle: 'Subtitle sample'
          },
          {
            icon: 'fa-exclamation-triangle',
            name: 'second',
            title: 'Sample title 2',
            subtitle: 'Subtitle sample'
          }
        ]
      }
    },
    keepAlive: {
      type: Boolean,
      default: true
    }
  },

  data () {
    return {
      currentStep: {},
      previousStep: {},
      nextButton: {},
      canContinue: false,
      finalStep: false
    }
  },

  computed: {
    enterAnimation () {
      if (this.currentStep.index < this.previousStep.index) {
        return 'animated quick fadeInLeft'
      } else {
        return 'animated quick fadeInRight'
      }
    },
    leaveAnimation () {
      if (this.currentStep.index > this.previousStep.index) {
        return 'animated quick fadeOutLeft'
      } else {
        return 'animated quick fadeOutRight'
      }
    }
  },

  methods: {
    onDataUpdate (stepData) {
      this.$emit('data-update', {step: currentStep, data: stepData})
    },
    isStepActive (index, step) {
      if (this.currentStep.index === index) {
        return 'activated'
      } else {
        return 'deactivated'
      }
    },

    activateStep (index, back = false) {
      if (this.steps[index]) {
        this.previousStep = this.currentStep
        this.currentStep = {
          name: this.steps[index].name,
          index: index
        }

        if (index + 1 === this.steps.length) {
          this.finalStep = true
        } else {
          this.finalStep = false
        }

        if (!back) {
          this.$emit('completed-step', this.previousStep)
        }
      }
      this.$emit('active-step', this.currentStep)
    },

    nextStep () {
      this.nextButton[this.currentStep.name] = true
      if (this.canContinue) {
        if (this.finalStep) {
          this.$emit('stepper-finished', this.currentStep)
        }
        let currentIndex = this.currentStep.index + 1

        this.activateStep(currentIndex)
      }
      this.canContinue = false
      this.$forceUpdate()
    },

    backStep () {
      this.$emit('clicking-back')
      let currentIndex = this.currentStep.index - 1
      if (currentIndex >= 0) {
        this.activateStep(currentIndex, true)
      }
    },

    proceed (payload) {
      this.canContinue = payload.value
    },

    changeNextBtnValue (payload) {
      this.nextButton[this.currentStep.name] = payload.nextBtnValue
      this.$forceUpdate()
    }
  },

  created () {
    // Initiate stepper
    this.activateStep(0)
    this.steps.forEach(step => {
      this.nextButton[step.name] = false
    })
  }
}
</script>

<style src="./HorizontalStepper.scss" scoped lang="scss"></style>