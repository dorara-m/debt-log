<template>
  <div id="app">

    <div class="app-content app-log">
      <h1>ログ</h1>
      <table>
        <thead>
          <tr>
            <th>No.</th>
            <th>いつ</th>
            <th>誰が</th>
            <th>何に</th>
            <th>いくら</th>
            <th>-</th>
            <th>-</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(mochi, index) in mochis" :key="index">
            <th>{{index}}</th>
            <td>{{mochi.date}}</td>
            <td>{{mochi.person}}</td>
            <td>{{mochi.label}}</td>
            <td>{{mochi.price}}</td>
            <td><button>編集</button></td>
            <td><button v-on:click="mochiSave(mochi)">×</button></td>
          </tr>
        </tbody>
      </table>

      <div class="modalWrap" v-if="isOpenModal">
        <div class="confirmModal">
          <p>本当に削除してもいいですか？</p>
          <button v-on:click="mochiRemove(true)">はい</button>
          <button v-on:click="mochiRemove(false)">いいえ</button>
        </div>
      </div>
    </div>

    <div class="app-content app-input">
      <h1>新規もち追加</h1>
      <p class="error" v-if="inputError">未入力の箇所があります</p>
      <form class="add-form" v-on:submit.prevent="mochiAdd">
        <p>いつ</p>
        <input type="date" ref="date">
        <p>誰が</p>
        <input type="text" ref="person">
        <p>何に</p>
        <input type="text" ref="label">
        <p>いくら払った</p>
        <input type="number" ref="price">円
        <button class="submit">記録</button>
      </form>
    </div>

  </div>
</template>

<script>
const STORAGE_KEY = 'mochi-vuejs-demo'
const mochiStorage = {
  fetch: function() {
    const mochis = JSON.parse(
      localStorage.getItem(STORAGE_KEY) || '[]'
    )
    mochis.forEach(function(mochi, index) {
      mochi.id = index
    })
    mochiStorage.uid = mochis.length
    return mochis
  },
  save: function(mochis) {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(mochis))
  }
}

export default {
  name: 'app',
  data: function(){
    return {
      mochis: [],
      inputError: false,
      tmpMochi: {},
      isOpenModal: false
    }
  },
  watch: {
    mochis: {
      handler: function(mochis) {
        mochiStorage.save(mochis)
      },
      deep: true
    }
  },
  created: function() {
    this.mochis = mochiStorage.fetch();
  },
  methods: {
    mochiAdd: function() {
      const date = this.$refs.date;
      const person = this.$refs.person;
      const label = this.$refs.label;
      const price = this.$refs.price;

      if (!date.value.length || !person.value.length || !label.value.length || !price.value.length) {
        this.inputError = true;
        return;
      }
      
      this.mochis.push ({
        id: mochiStorage.uid++,
        date: date.value,
        person: person.value,
        label: label.value,
        price: price.value
      });

      date.value = '';
      person.value = '';
      label.value = '';
      price.value = '';
      this.inputError = false;
    },
    mochiSave: function(item) {
      this.tmpMochi = item;
      this.isOpenModal = true;
    },
    mochiRemove: function(isYes) {
      if (isYes) {
        const index = this.mochis.indexOf(this.tmpMochi);
        this.mochis.splice(index, 1);
      }
      this.isOpenModal = false;
    }
  }
}
</script>

<style lang="scss" scoped>
.app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.app-content {
  max-width: 800px;
  margin: 0 auto;
  margin-bottom: 100px;
  h1 {
    text-align: center;
  }
}

.app-input {
  section {
    margin-top: 30px;
  }
  .error {
    color: red;
  }
  .submit {
    margin-top: 30px;
    width: 200px;
    font-size: 20px;
  }
}

.app-log {
  table {
    width: 100%;
    border-collapse: collapse;
  }
  td, th {
    border: 1px solid #444;
    text-align: center;
  }
}
</style>
