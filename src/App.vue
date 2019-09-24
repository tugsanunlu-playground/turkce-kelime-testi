<template>
  <div id="app">
    <div class="container">
      <div class="d-flex flex-column justify-content-start align-items-center">
        <div v-if="!isStart" class="jumbotron shadow-sm w-100">
          <h3>Türkçe Kelime Testi</h3>
          <p>Yazımı sıklıkla karıştırılan Türkçe kelimelerle ilgili on soruluk bir test.</p>

          <button @click="startTest" class="btn btn-sm btn-primary" :disabled="isLoader">
            <span v-if="!isLoader">BAŞLA</span>
            <span v-else class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span>
          </button>
        </div>

        <div v-if="isStart && !isResult" class="jumbotron shadow-sm d-flex flex-column w-100">
          <div>
            <h5>Doğru yazımı seçiniz.</h5>
          </div>

          <div v-for="(question, index) in questions" :key="question.id">
            <div class="form-group" v-show="index === questionIndex">
              <h5>
                <span class="badge badge-light">{{ index+1 }} / {{ questions.length }}</span>
              </h5>

              <div class="d-flex flex-column mt-2 mb-3">
                <div
                  class="form-check form-check-inline my-1"
                  :class="{'order-1': randomOrderValue === 1 }"
                >
                  <input
                    class="form-check-input"
                    type="radio"
                    :name="index"
                    :id="question.dogrukelime"
                    :value="true"
                    v-model="answers[index]"
                  />
                  <label
                    class="form-check-label"
                    :for="question.dogrukelime"
                  >{{ question.dogrukelime }}</label>
                </div>
                <div class="form-check form-check-inline my-1">
                  <input
                    class="form-check-input"
                    type="radio"
                    :name="index"
                    :id="question.yanliskelime"
                    :value="false"
                    v-model="answers[index]"
                  />
                  <label
                    class="form-check-label"
                    :for="question.yanliskelime"
                  >{{ question.yanliskelime }}</label>
                </div>
              </div>

              <button
                v-if="questionIndex < questions.length -1"
                @click="nextQuestion"
                class="btn btn-sm btn-primary align-self-start"
                :disabled="answers[index] == undefined"
              >SONRAKİ SORU</button>

              <button
                v-else
                @click="finishTest"
                class="btn btn-sm btn-primary align-self-start"
                :disabled="answers[index] == undefined"
              >BİTİR</button>
            </div>
          </div>

          <div class="mt-2">
            <small class="bg-light p-2">
              <font-awesome-icon :icon="'exclamation-triangle'" /> Uzun okunan heceler ":" ile gösterilir.
            </small>
          </div>
        </div>

        <div v-if="isResult" class="jumbotron shadow-sm w-100">
          <h3>Türkçe Kelime Testi Sonuçları</h3>
          <p>
            <strong>{{ questions.length }}</strong> sorudan
            <strong>{{ countCorrect() }}</strong> tanesine doğru cevap verdiniz.
          </p>

          <div class="d-flex flex-column justify-content-between">
            <div
              v-for="(question, index) in questions"
              :key="question.id"
              class="alert shadow-sm"
              :class="checkStatus(index) ? 'alert-success' : 'alert-danger'"
            >
              <font-awesome-icon
                :icon="checkStatus(index) ? 'check' : 'times'"
                class="mr-2 status-icon"
              />
              <div class="text-success">
                <strong>Doğru Yazım:</strong>
                {{ question.dogrukelime }}
              </div>
              <div class="text-danger">
                <strong>Yanlış Yazım:</strong>
                {{ question.yanliskelime }}
              </div>
            </div>
          </div>

          <button @click="startTest" class="btn btn-sm btn-primary">YENİDEN BAŞLA</button>
        </div>
        <div>
          <small>
            <a
              href="http://github.com/tugsanunlu/turkce-kelime-testi"
              target="_blank"
              class="text-white"
            >
              <font-awesome-icon :icon="['fab', 'github']" />/tugsanunlu/turkce-kelime-testi
            </a>
          </small>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      questions: [],
      answers: [],
      questionIndex: 0,
      questionNumber: 10,
      randomOrderValue: 0,
      isLoader: false,
      isStart: false,
      isResult: false,
    };
  },
  methods: {
    resetInitialData() {
      Object.assign(this.$data, this.$options.data.call(this));
    },
    startTest() {
      this.resetInitialData();
      this.isLoader = true;
      this.getQuestions();
      this.randomOrder();
    },
    async getQuestions() {
      for (let i = 1; i <= this.questionNumber / 2; i += 1) {
        await fetch('https://sozluk.gov.tr/icerik').then(async response => {
          const data = await response.json();
          this.questions = [...this.questions, ...data.syyd];
        });
      }
      this.isStart = true;
    },
    nextQuestion() {
      this.questionIndex += 1;
      this.randomOrder();
    },
    randomOrder() {
      this.randomOrderValue = Math.round(Math.random());
    },
    checkStatus(index) {
      return this.answers[index] === true;
    },
    finishTest() {
      this.isResult = true;
    },
    countCorrect() {
      return this.answers.filter(filter => filter === true).length;
    },
  },
};
</script>

<style scoped>
.container {
  max-width: 600px;
}
.status-icon {
  position: absolute;
  right: 10px;
  font-size: 50px;
  opacity: 0.1;
}
.fade-enter-active {
  transition: opacity 0.5s;
}
.fade-enter {
  opacity: 0.3;
}
</style>