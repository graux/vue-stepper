<template>
    <div class="stepper-box">
        <div class="top">
            <div class="divider-line"></div>
            <div class="steps-wrapper">
                <template v-for="(step, index) in steps">
                    <div :class="['step', isStepActive(index, step)]">
                        <div class="circle">
                            <i class="material-icons md-18">
                                {{ (step.completed) ? 'done' : step.icon }}
                            </i>
                        </div>
                        <div class="step-title">
                            <h4>{{step.title}}</h4>
                            <h5 class="step-subtitle">{{step.subtitle}}</h5>
                        </div>
                    </div>
                </template>
            </div>
        </div>
        <div class="content">
            <transition
                    :enter-active-class="enterAnimation"
                    :leave-active-class="leaveAnimation"
                    mode="out-in"
            >
                <keep-alive>
                    <component :is="steps[currentStep.index].component" :clickedNext="nextButton[currentStep.name]" @can-continue="proceed"></component>
                </keep-alive>
            </transition>
        </div>
        <div :class="['bottom', (currentStep.index > 0) ? '' : 'only-next']">
            <div v-if="currentStep.index > 0" class="stepper-button previous" @click="backStep()">
                <i class="material-icons">keyboard_arrow_left</i>
                <span>Atrás</span>
            </div>
            <div :class="['stepper-button next', !canContinue ? 'deactivated' : '']" @click="nextStep()">
                <span>{{ (finalStep) ? 'Finalizar' : 'Siguiente' }}</span>
                <i class="material-icons">keyboard_arrow_right</i>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        props: {
            steps: {
                type: Array,
                default: function () {
                    return [
                        {
                            icon: 'mail',
                            name: 'first',
                            title: 'Sample title 1',
                            subtitle: 'Subtitle sample'

                        },
                        {
                            icon: 'report_problem',
                            name: 'second',
                            title: 'Sample title 2',
                            subtitle: 'Subtitle sample'
                        }
                    ]
                }
            }
        },
        data() {
            return {
                currentStep: {},
                previousStep: {},
                nextButton: {},
                canContinue: false,
                finalStep: false
            }
        },
        computed: {
            enterAnimation() {
                if (this.currentStep.index < this.previousStep.index) {
                    return 'animated quick fadeInLeft'
                } else {
                    return 'animated quick fadeInRight'
                }
            },
            leaveAnimation() {
                if (this.currentStep.index > this.previousStep.index) {
                    return 'animated quick fadeOutLeft'
                } else {
                    return 'animated quick fadeOutRight'
                }
            }
        },
        methods: {
            isStepActive(index, step) {
                if (this.currentStep.index === index) {
                    return 'activated'
                } else {
                    return 'deactivated'
                }
            },
            activateStep(index, back = false) {
                if(this.steps[index]) {
                    this.previousStep = this.currentStep;
                    this.currentStep = {
                        name: this.steps[index].name,
                        index: index
                    };

                    if(index + 1 === this.steps.length) {
                        this.finalStep = true;
                    } else {
                        this.finalStep = false;
                    }

                    if(!back) {
                        this.$emit('completed-step', this.previousStep);
                    }
                }
                this.$emit('active-step', this.currentStep);
            },
            nextStep() {
                if (this.canContinue) {
                    if(this.finalStep) {
                        this.$emit('stepper-finished', this.currentStep);
                    }
                    let currentIndex = this.currentStep.index + 1;
                    this.activateStep(currentIndex);

                }
                this.nextButton[this.currentStep.name] = true;
                this.$forceUpdate();
            },
            backStep() {
                this.$emit('clicking-back');
                let currentIndex = this.currentStep.index - 1;
                if (currentIndex >= 0) {
                    this.activateStep(currentIndex, true);
                }
            },
            proceed(payload) {
                this.canContinue = payload.value;
            }
        },
        created() {
            // Initiate stepper
            this.activateStep(0);
            this.steps.forEach( (step)=> {
                this.nextButton[step.name] = false;
            });
        }
    }
</script>

<style src="./HorizontalStepper.scss" scoped lang="scss"></style>