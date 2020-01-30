<template>
  <div class="container">
    <nav class="navbar navbar-dark bg-dark rounded-sm">
      <span class="navbar-brand">{{ msg }}</span>
      <div class="form-inline">
        <button class="btn btn-outline-danger" v-on:click="resetGrades">Törlés</button>
      </div>
    </nav>
    <div class="row">
      <div class="col-sm-auto col-lg-8">
        <form>
          <div v-for="(grade, index) in grades" v-bind:key="index">
            <div class="form-row">
              <div class="col">
                <div class="input-group">
                  <div class="input-group-prepend">
                    <div class="input-group-text">{{ index+1 }}.</div>
                  </div>
                  <input v-model.number="grade.credit" type="number"
                    :class="[(grade.credit >= 1 && grade.credit <= 15 || !grade.credit && grade.credit !== 0) ? '' : 'is-invalid']"
                    name="grades[][credit]" class="form-control"
                    placeholder="Kredit" v-on:input="calc" v-on:blur="removeGrade">
                </div>
              </div>
              <div class="col">
                <input v-model.number="grade.grade" type="number"
                  :class="[(grade.grade >= 1 && grade.grade <= 5 || !grade.grade && grade.grade !== 0) ? '' : 'is-invalid']"
                  name="grades[][grade]" class="form-control"
                  placeholder="Jegy" v-on:focus="addGrade(index, grade)"
                  v-on:blur="removeGrade" v-on:input="calc">
              </div>
            </div>
          </div>
        </form>
      </div>
      <div class="col-sm-auto col-lg-4">
        <div class="card border-dark">
          <div class="card-body text-dark">
            <h5 class="card-title">Átlagok</h5>
          </div>
          <ul class="list-group list-group-flush">
            <li class="list-group-item">Hagyományos átlag: <b>{{ averages.ha }}</b></li>
            <li class="list-group-item">Súlyozott tanulmányi átlag: <b>{{ averages.sta }}</b></li>
            <li class="list-group-item">Kreditindex: <b>{{ averages.ki }}</b></li>
            <li class="list-group-item">Korrigált kreditindex: <b>{{ averages.kki }}</b></li>
            <li class="list-group-item">Felvett kredit: <b>{{ stats.fk }}</b></li>
            <li class="list-group-item">Teljesített kredit: <b>{{ stats.tk }}</b></li>
            <li class="list-group-item">Felvett tárgyak: <b>{{ stats.ft }}</b></li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import 'bootstrap'
import Vue from 'vue'

export default {
  name: 'Calculator',
  props: {
    msg: String
  },
  data: () => ({
    averages: {
      ha: 0.0,
      sta: 0.0,
      ki: 0.0,
      kki: 0.0
    },
    stats: {
      fk: 0,
      tk: 0,
      ft: 0
    },
    grade: {
      credit: null,
      grade: null
    },
    grades: []
  }),
  methods: {
    resetGrades: function () {
      this.grades = []
      this.startGrade()
      this.resetStats()
    },
    addGrade: function (index, grade) {
      this.grades.push(Vue.util.extend({}, this.grade))
      // this.removeGrade(index, grade)
    },
    startGrade: function () {
      this.grades.push(Vue.util.extend({}, this.grade))
    },
    removeGrade: function () {
      let index = this.grades.length
      while (!(this.grades[index - 1].credit || this.grades[index - 1].grade) && index !== 0) {
        Vue.delete(this.grades, index)
        index--
      }
    },
    resetStats: function () {
      this.averages.ha = 0.0
      this.averages.sta = 0.0
      this.averages.ki = 0.0
      this.averages.kki = 0.0
      this.stats.fk = 0
      this.stats.tk = 0
      this.stats.ft = 0
    },
    calc: function () {
      this.resetStats()
      this.grades.forEach((grade, index) => {
        if (grade.grade) {
          this.stats.ft++
        }
      })

      this.grades.forEach((grade, index) => {
        if (grade.grade && grade.grade) {
          this.averages.ha += grade.grade
          this.averages.sta += (grade.grade * grade.credit)
          this.stats.fk += grade.credit
          if (grade.grade !== 1) {
            this.stats.tk += grade.credit
          }
        }
      })

      if (this.stats.ft !== 0) {
        this.averages.ha = (this.averages.ha / this.stats.ft).toFixed(2).replace('.', ',')
        this.averages.ki = (this.averages.sta / 30).toFixed(2).replace('.', ',')
        this.averages.kki = ((this.averages.sta / 30) * (this.stats.tk / this.stats.fk)).toFixed(2).replace('.', ',')
        this.averages.sta = (this.averages.sta / this.stats.tk).toFixed(2).replace('.', ',')
      }
    }
  },
  mounted: function () {
    this.startGrade()
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
@import "~bootstrap/scss/bootstrap";

.rounded-sm {
  border-top-left-radius: 0 !important;
  border-top-right-radius: 0 !important;
}
.row {
  margin-top: 1em;
  margin-bottom: 1em;
}
.form-row {
  margin-bottom: 1em
}
.col-sm-auto {
  width: 100%;
}
</style>
