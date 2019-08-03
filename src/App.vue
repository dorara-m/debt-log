<template>
  <div class="app" id="app">
  <!-- <div class="app" id="app"> -->
    <main :class="{blur: isOpenModal}">
      <header>
        <div class="container">
          <h1 class="app-title">Mochi-Mochi</h1>
        </div>
      </header>

      <div class="app-select">
        <div class="container flex">
          <select v-model="current">
            <option v-for="(user, index) in users" :key="index" :value="user">{{user}}</option>
          </select>
        </div>
      </div>

      <div class="app-table">
        <div class="container">
          <div class="app-table-date" v-for="(date, index) in dateList" :key="index">
            <h3>{{date}}</h3>
            <table>
              <tbody>
                <tr v-for="(mochi, index) in currentDateMochi(date)" :key="index">
                  <td>{{mochi.person}}</td>
                  <td>{{mochi.label}}</td>
                  <td>{{mochi.price}}</td>
                  <td><button v-on:click="openModal(mochi)">編集</button></td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>

      <div class="app-input">
        <div class="container">
          <h2>新規もち追加</h2>
          <p class="error" v-if="inputError">未入力の箇所があります</p>
          <form class="add-form" v-on:submit.prevent="mochiAdd">
            <div class="flex">
              ¥<input type="number" ref="price" placeholder="いくら">
            </div>
            <div>
              <input type="text" ref="person" placeholder="誰が">
            </div>
            <div>
              <input type="date" ref="date" placeholder="いつ">
            </div>
            <div>
              <input type="text" ref="label" placeholder="何に">
            </div>
            <button class="submit">記録</button>
          </form>
          <div class="debug"><button v-on:click="mochiAdd_demo">デモデータ追加</button></div>
        </div>
      </div>
    </main>

    <div class="modalWrap" v-if="isOpenModal">
      <div class="modal">
        <div class="container">
          <div class="editModal">
            <h2>Edit</h2>
            <p class="error" v-if="inputError">未入力の箇所があります</p>
            <button class="close" v-on:click="closeModal">☓</button>
            <form class="edit-form">
              <div class="inputWrap price">
                ¥<input type="number" ref="price_edit" :value="tmpMochi.price" placeholder="いくら">
              </div>
              <div class="inputWrap">
                <input type="date" ref="date_edit" :value="tmpMochi.date" placeholder="いつ">
              </div>
              <div class="inputWrap">
                <input type="text" ref="label_edit" :value="tmpMochi.label" placeholder="何に">
              </div>
              <div class="inputWrap">
                <input type="text" ref="person_edit" :value="tmpMochi.person" placeholder="誰が">
              </div>
            </form>
            <div class="btnArea">
              <button v-on:click="isConfirm = true">delete</button>
              <button class="submit" v-on:click="mochiEdit">update</button>
            </div>
            <div v-if="isConfirm" class="confirmArea">
              <p>本当に削除してもいいですか？</p>
              <button v-on:click="mochiRemove">はい</button>
              <button v-on:click="isConfirm = false">いいえ</button>
            </div>
          </div>
        </div>
      </div>
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
      tmpMochi: {},
      isConfirm: false,
      inputError: false,
      isOpenModal: false,
      scrollPosition: 0,
      current: '全員'
    }
  },
  computed: {
    users: function() {
      let persons = this.mochis.map(mochi => mochi.person);
      persons.unshift('全員');
      return persons.filter((x, i, self) => self.indexOf(x) === i);
    },
    computedMochi: function() {
      return this.mochis.filter((el) => {
        return this.current === '全員' ? true : this.current === el.person;
      }, this);
    },
    dateList: function() {
      const date = this.computedMochi.map(el => el.date);
      const uniqueDate = date.filter((x, i, self) => self.indexOf(x) === i);
      return uniqueDate.sort((a,b) => new Date(b) - new Date(a));
    },
    currentDateMochi: function() {
      return function(date) {
        return this.computedMochi.filter((el) => el.date === date);
      }
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
    openModal: function(item) {
      this.tmpMochi = item;
      this.isOpenModal = true;
      this.initNoScroll();
    },
    closeModal: function() {
      this.isOpenModal = false;
      this.destoryNoScroll();
    },
    initNoScroll() {
      this.scrollPosition = window.pageYOffset;
      const $body = document.body;
      $body.classList.add('isNoScroll');
      $body.style.setProperty('top', `-${this.scrollPosition}px`);
    },
    destoryNoScroll() {
      const $body = document.body;
      $body.classList.remove('isNoScroll');
      $body.style.removeProperty('top');
      window.scrollTo(0, this.scrollPosition);
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
      this.mochis = mochiStorage.fetch();
      this.inputError = false;
      this.closeModal();
    },
    mochiRemove: function() {
      const index = this.mochis.indexOf(this.tmpMochi);
      this.mochis.splice(index, 1);
      this.isConfirm = false;
      this.closeModal();
    }
  }
}
</script>

<style lang="scss" scoped>
@import url(./assets/_reset.css);
body {
  &.isNoScroll {
    overflow: hidden;
    height: 100%;
    width: 100%;
    position: fixed;
  }
}
.app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin: 0 auto;
  position: relative;
  padding: 50px 0;
}
.container {
  padding: 0 20px;
  @media print, screen and (min-width: 767px) {
    max-width: 500px;
    margin: 0 auto;
  }
}
input, button, textarea, select {
	margin: 0;
	padding: 0;
	background: none;
	border: none;
	border-radius: 0;
	outline: none;
	-webkit-appearance: none;
	-moz-appearance: none;
	appearance: none;
}
main {
  &.blur {
    opacity: .3;
  }
}
header {
  padding: 25px;
  .app-title {
    font-size: 40px;
    font-weight: bold;
    text-align: center;
  }
}
.app-select {
  margin-top: 60px;
  > .flex {
    display: flex;
    justify-content: flex-end;
    select {
      font-size: 14px;
      width: 120px;
    }
  }
}
.app-table {
  margin-top: 50px;
  &-date {
    margin-top: 20px;
    > h3 {
      font-size: 20px;
      font-weight: bold;
      margin-bottom: 5px;
    }
    table {
      width: 100%;
      td, th {
        border: 1px solid #444;
        text-align: center;
      }
    }
  }
}

.modalWrap {
  position: fixed;
  background-color: rgba($color: #000, $alpha: .4);
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  > .modal {
    position: absolute;
    bottom: 0;
    left: 0;
    width: 100%;
    background-color: #fff;
  }
  .editModal {
    padding: 40px 0;
    position: relative;
    font-weight: bold;
    > h2 {
      font-size: 24px;
      text-align: center;
      margin-bottom: 20px;
    }
    .close {
      position: absolute;
      top: 20px;
      left: 0;
    }
    .inputWrap {
      border-bottom: 1px solid #bbb;
      padding-bottom: 3px;
      margin-bottom: 15px;
      font-size: 13px;
      > input {
        font-weight: inherit;
        font-size: inherit;
        display: inline-block;
      }
      &.price {
        display: flex;
        justify-content: space-between;
        font-size: 18px;
        padding-bottom: 5px;
        margin-bottom: 25px;
        > input {
          text-align: right;
        }
      }
    }
    .btnArea {
      margin-top: 50px;
      width: 210px;
      margin-right: auto;
      margin-left: auto;
      display: flex;
      justify-content: space-around;
      > button {
        border: 1px solid #999;
        font-size: 16px;
        padding: 6px 10px;
        font-weight: inherit;
      }
    }
  }
}

.app-input {
  margin-top: 50px;
  .error {
    color: red;
  }
  .submit {
    margin-top: 30px;
    width: 200px;
    font-size: 20px;
  }
}
</style>
