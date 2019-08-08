<template>
  <div class="app" id="app">
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

      <div class="app-data">
        <div class="container">
          <div class="app-data-date" v-for="(date, index) in dateList" :key="index">
            <h3>{{date}}</h3>
            <ul class="app-data-list">
              <li v-for="(mochi, index) in currentDateMochi(date)" :key="index">
                <button v-on:click="openEditModal(mochi)">
                  <div class="right">
                    <div class="person">{{mochi.person}}</div>
                    <div class="label">{{mochi.label}}</div>
                  </div>
                  <div class="price">¥{{mochi.price}}</div>
                </button>
              </li>
            </ul>
          </div>
        </div>
      </div>
      
      <button class="app-new" v-on:click="openModal">新規</button>
    </main>

    <div class="modalWrap" v-if="isOpenModal">
      <div class="modal">
        <div class="modalContents editModal" v-if="isEdit" :class="{confirmOpen : isConfirm}">
          <div class="container">
            <h2>Edit</h2>
            <p class="error" v-if="inputError">未入力の箇所があります</p>
            <button class="close" v-on:click="closeModal"><span></span></button>
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
          </div>
          <div v-if="isConfirm" class="confirmArea">
            <p>本当に削除してもいいですか？</p>
            <div class="btns">
              <button v-on:click="mochiRemove">Yes</button>
              <button v-on:click="isConfirm = false">No</button>
            </div>
          </div>
        </div>
        <div v-else class="modalContents addModal">
          <div class="container">
            <h2>New</h2>
            <p class="error" v-if="inputError">未入力の箇所があります</p>
            <button class="close" v-on:click="closeModal"><span></span></button>
            <form class="edit-form">
              <div class="inputWrap price">
                ¥<input type="number" ref="price" placeholder="いくら">
              </div>
              <div class="inputWrap">
                <input type="date" ref="date" :value="today" placeholder="いつ">
              </div>
              <div class="inputWrap">
                <input type="text" ref="label" placeholder="何に">
              </div>
              <div class="inputWrap">
                <input type="text" ref="person" placeholder="誰が">
              </div>
            </form>
            <div class="btnArea">
              <button v-on:click="mochiAdd_demo">Demo</button>
              <button class="submit" v-on:click="mochiAdd">Add</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import firebase from 'firebase/app'
import "firebase/database";

export default {
  name: 'app',
  data: function(){
    return {
      mochisRef: null,
      mochis: [],
      tmpMochi: {},
      inputError: false,
      isOpenModal: false,
      isEdit: false,
      isConfirm: false,
      scrollPosition: 0,
      current: '全員'
    }
  },
  created: function() {
    this.mochisRef = firebase.database().ref('/mochi-log');
    var _this = this;
    this.mochisRef.on('value', function(snapshot) {
      const mochisObject = snapshot.val();
      _this.mochis = Object.keys(mochisObject).map(key => Object.assign({id: key}, mochisObject[key]));
    });
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
    },
    today: function() {
      const dt = new Date();
      const y = dt.getFullYear();
      const m = ("00" + (dt.getMonth()+1)).slice(-2);
      const d = ("00" + dt.getDate()).slice(-2);
      return `${y}-${m}-${d}`;
    }
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
      
      this.mochisRef.push ({
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
      this.closeModal();
    },
    mochiAdd_demo: function() {
      this.mochisRef.push ({
        date: '2019-07-01',
        person: 'サンプルさん',
        label: 'お茶',
        price: '3000'
      });
      this.closeModal();
    },
    openModal: function() {
      this.isEdit = false;
      this.isOpenModal = true;
      this.initNoScroll();
    },
    openEditModal: function(item) {
      this.isEdit = true;
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

      const datas = {
        date: date.value,
        person: person.value,
        label: label.value,
        price: price.value
      };
      const updates = {};
      updates[tmpId] = datas;
      this.mochisRef.update(updates);
      this.inputError = false;
      this.closeModal();
    },
    mochiRemove: function() {
      const index = this.tmpMochi.id;
      this.mochisRef.child(index).remove();
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
  letter-spacing: 0.02em;
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
	-webkit-appearance: none;
	-moz-appearance: none;
	appearance: none;
  text-align: left;
  letter-spacing: inherit;
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
  margin-top: 30px;
  > .flex {
    display: flex;
    justify-content: flex-end;
    select {
      font-size: 14px;
      width: 120px;
    }
  }
}
.app-data {
  margin-top: 20px;
  &-date {
    margin-top: 25px;
    > h3 {
      font-size: 20px;
      font-weight: bold;
      margin-bottom: 5px;
    }
  }
  &-list {
    li {
      margin-bottom: 8px;
      > button {
        width: 100%;
        border: 1px solid #bbb;
        padding: 5px 8px;
        display: flex;
        justify-content: space-between;
        align-items: flex-end;
        cursor: pointer;
        > .right {
          > .person {
            font-size: 12px;
            color: #555;
            margin-bottom: 3px;
          }
          > .label {
            font-size: 14px;
            font-weight: bold;
          }
        }
        > .price {
          font-size: 18px;
          font-weight: bold;
        }
      }
    }
  }
}

.app-new {
  position: fixed;
  bottom: 20px;
  right: 30px;
  font-size: 24px;
  border: 1px solid #444;
  background-color: #fff;
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
  .close {
    cursor: pointer;
    position: absolute;
    top: 20px;
    left: 20px;
    width: 20px;
    height: 20px;
    span {
      position: absolute;
      top: 50%;
      transform: translateY(-50%);
      left: 0;
      width: 100%;
      height: 2px;;
      &:before, &:after {
        content: '';
        position: absolute;
        height: 100%;
        width: 100%;
        background-color: #444;
      }
      &:before {
        transform: rotate(45deg);
      }
      &:after {
        transform: rotate(-45deg);
      }
    }
  }
  .modalContents {
    padding: 40px 0;
    position: relative;
    font-weight: bold;
    &.confirmOpen::before {
      content: '';
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background-color: rgba($color: #000, $alpha: .7);
      z-index: 2;
    }
    h2 {
      font-size: 24px;
      text-align: center;
      margin-bottom: 20px;
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

    .confirmArea {
      position: absolute;
      bottom: 0;
      z-index: 2;
      padding: 20px;
      background-color: #fff;
      width: 100%;
      box-sizing: border-box;
      text-align: center;
      .btns {
        margin-top: 20px;
        display: flex;
        width: 110px;
        margin-right: auto;
        margin-left: auto;
        justify-content: space-between;
        > button {
          border: 1px solid #999;
          font-size: 16px;
          padding: 6px 10px;
          font-weight: inherit;
        }
      }
    }
  }
}

.error {
  color: red;
  font-size: 14px;
  margin-bottom: 10px;
  text-align: center;
  font-weight: bold;
}

</style>
