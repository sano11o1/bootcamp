<template lang="pug">
.page-body
  .container(v-if='question === null || currentUser === null')
    .empty
      .fas.fa-spinner.fa-pulse
      | ロード中
  .container.is-lg(v-else)
    questionEdit(
      :question='question',
      :answerCount='answerCount',
      :isAnswerCountUpdated='isAnswerCountUpdated',
      :currentUser='currentUser'
    )
    a#comments.a-anchor
    answers(
      :questionId='questionId',
      :questionUser='question.user',
      :currentUser='currentUser',
      @updateAnswerCount='updateAnswerCount',
      @solveQuestion='solveQuestion',
      @cancelSolveQuestion='cancelSolveQuestion'
    )
</template>
<script>
import QuestionEdit from './question-edit.vue'
import Answers from './answers.vue'

export default {
  components: {
    questionEdit: QuestionEdit,
    answers: Answers
  },
  props: {
    currentUserId: { type: String, required: true },
    questionId: { type: String, required: true }
  },
  data() {
    return {
      question: null,
      currentUser: null,
      answerCount: 0,
      isAnswerCountUpdated: false
    }
  },
  created() {
    this.fetchQuestion(this.questionId)
    this.fetchUser(this.currentUserId)
  },
  methods: {
    fetchQuestion(id) {
      fetch(`/api/questions/${id}.json`, {
        method: 'GET',
        headers: {
          'X-Requested-With': 'XMLHttpRequest',
          'X-CSRF-Token': this.token()
        },
        credentials: 'same-origin'
      })
        .then((response) => {
          return response.json()
        })
        .then((question) => {
          this.question = question
        })
        .catch((error) => {
          console.warn('Failed to parsing', error)
        })
    },
    fetchUser(id) {
      fetch(`/api/users/${id}.json`, {
        method: 'GET',
        headers: {
          'X-Requested-With': 'XMLHttpRequest'
        },
        credentials: 'same-origin',
        redirect: 'manual'
      })
        .then((response) => {
          return response.json()
        })
        .then((user) => {
          this.currentUser = user
        })
        .catch((error) => {
          console.warn('Failed to parsing', error)
        })
    },
    solveQuestion(answer) {
      this.question.correct_answer = answer
    },
    cancelSolveQuestion() {
      this.question.correct_answer = null
    },
    token() {
      const meta = document.querySelector('meta[name="csrf-token"]')
      return meta ? meta.getAttribute('content') : ''
    },
    updateAnswerCount(count) {
      this.answerCount = count
      this.isAnswerCountUpdated = true
    }
  }
}
</script>
