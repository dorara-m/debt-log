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
            <td><button v-on:click="openModal(mochi, true)">編集</button></td>
            <td><button v-on:click="openModal(mochi, false)">×</button></td>
          </tr>
        </tbody>
      </table>

      <div class="modalWrap" v-if="isOpenModal">
        <div v-if="isEdit" class="editModal">
          <p>編集画面</p>
          <p class="error" v-if="inputError">未入力の箇所があります</p>
          <form class="edit-form" v-on:submit.prevent="mochiEdit">
            <p>いつ</p>
            <input type="date" ref="date_edit" :value="tmpMochi.date">
            <p>誰が</p>
            <input type="text" ref="person_edit" :value="tmpMochi.person">
            <p>何に</p>
            <input type="text" ref="label_edit" :value="tmpMochi.label">
            <p>いくら払った</p>
            <input type="number" ref="price_edit" :value="tmpMochi.price">円
            <button class="submit">更新</button>
          </form>
          <button v-on:click="closeModal">キャンセル</button>
        </div>
        <div v-else class="confirmModal">
          <p>本当に削除してもいいですか？</p>
          <button v-on:click="mochiRemove">はい</button>
          <button v-on:click="closeModal">いいえ</button>
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
      <div class="debug"><button v-on:click="mochiAdd_demo">デモデータ追加</button></div>
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
    mochiAdd_demo: function() {
      this.mochis.push ({
        id: mochiStorage.uid++,
        date: '2019-06-01',
        person: 'サンプル',
        label: 'サンプルだよ',
        price: '3000'
      });
    },
    openModal: function(item, isEdit) {
      this.tmpMochi = item;
      if (isEdit) {
        this.isEdit = true;
      }
      this.isOpenModal = true;
    },
    closeModal: function() {
      this.isOpenModal = false;
    },
    mochiEdit: function() {
      const tmpId = this.tmpMochi.id;
      const date = this.$refs.date_edit;
      const person = this.$refs.person_edit;
      const label = this.$refs.label_edit;
      const price = this.$refs.price_edit;

      if (!date.value.length || !person.value.length || !label.value.length || !price.value.length) {
        this.inputError = true;
        return;
      }

      //同じidのもちのindexを特定
      const mochis = this.mochis;
      const targetIndex = mochis.findIndex(mochi => mochi.id === tmpId);
      // 上書き
      mochis[targetIndex] = {
        date: date.value,
        person: person.value,
        label: label.value,
        price: price.value
      };
      mochiStorage.save(mochis);

      this.inputError = false;
      this.isOpenModal = false;
    },
    mochiRemove: function() {
      const index = this.mochis.indexOf(this.tmpMochi);
      this.mochis.splice(index, 1);
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
