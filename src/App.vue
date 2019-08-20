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
                    <div class="person">
                      <span>{{mochi.personA}}</span>
                      <span>→</span>
                      <span>{{mochi.personB}}</span>
                    </div>
                    <div class="label">{{mochi.label}}</div>
                  </div>
                  <div class="price">¥{{mochi.price.replace(/(\d)(?=(\d{3})+(?!\d))/g, '$1,')}}</div>
                </button>
              </li>
            </ul>
          </div>
        </div>
      </div>

      <div class="app-new">
        <div class="container">
          <button v-on:click="openAddModal"><span></span></button>
        </div>
      </div>
    </main>

    <div class="modalWrap" v-if="isOpenModal">
      <div class="modal" :class="{confirmOpen : isConfirm}">
        <div class="container">
          <button class="close" v-on:click="closeModal"><span></span></button>
          <div class="modalContents editModal" v-if="isEdit">
            <h2>Edit</h2>
            <p class="error" v-if="inputError">未入力の箇所があります</p>
            <form class="edit-form">
              <div class="inputWrap price">
                ¥<input type="number" ref="price_edit" :value="tmpMochi.price" placeholder="いくら">
              </div>
              <div class="inputWrap">
                <input type="date" ref="date_edit" :value="tmpMochi.date" placeholder="いつ">
              </div>
              <div class="inputWrap person">
                <input type="text" ref="personA_edit" :value="tmpMochi.personA" placeholder="誰">
                <span>が</span>
                <input type="text" ref="personB_edit" :value="tmpMochi.personB" placeholder="誰">
                <span class="rent">に借りた</span>               
              </div>
              <div class="inputWrap">
                <input type="text" ref="label_edit" :value="tmpMochi.label" placeholder="何に">
              </div>
            </form>
            <div class="btnArea">
              <button class="c-btn-gray" v-on:click="isConfirm = true">Delete</button>
              <button class="c-btn-black" v-on:click="mochiEdit">Update</button>
            </div>
          </div>
          <div class="modalContents addModal" v-else>
            <h2>New</h2>
            <p class="error" v-if="inputError">未入力の箇所があります</p>
            <form class="edit-form">
              <div class="inputWrap price">
                ¥<input type="number" ref="price" placeholder="いくら">
              </div>
              <div class="inputWrap">
                <input type="date" ref="date" :value="today" placeholder="いつ">
              </div>
              <div class="inputWrap person">
                <input type="text" ref="personA" placeholder="誰">
                <span>が</span>
                <input type="text" ref="personB" placeholder="誰">
                <span class="rent">に借りた</span>
              </div>
              <div class="inputWrap">
                <input type="text" ref="label" placeholder="用途">
              </div>
            </form>
            <div class="btnArea">
              <button class="c-btn-gray" v-on:click="mochiAdd_demo">Demo</button>
              <button class="c-btn-black" v-on:click="mochiAdd">Add</button>
            </div>
          </div>
        </div>
        <div v-if="isConfirm" class="confirmArea">
          <p>本当に削除してもいいですか？</p>
          <div class="btns">
            <button class="c-btn-gray" v-on:click="isConfirm = false">No</button>
            <button class="c-btn-red" v-on:click="mochiRemove">Yes</button>
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
      let persons = this.mochis.map(mochi => mochi.personA);
      persons.unshift('全員');
      return persons.filter((x, i, self) => self.indexOf(x) === i);
    },
    computedMochi: function() {
      return this.mochis.filter((el) => {
        return this.current === '全員' ? true : this.current === el.personA;
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
      const personA = this.$refs.personA;
      const personB = this.$refs.personB;
      const label = this.$refs.label;
      const price = this.$refs.price;

      if (!date.value.length || !personA.value.length || !personB.value.length || !label.value.length || !price.value.length) {
        this.inputError = true;
        return;
      }

      this.mochisRef.push ({
        date: date.value,
        personA: personA.value,
        personB: personB.value,
        label: label.value,
        price: price.value
      });

      date.value = '';
      personA.value = '';
      personB.value = '';
      label.value = '';
      price.value = '';
      this.inputError = false;
      this.closeModal();
    },
    mochiAdd_demo: function() {
      this.mochisRef.push ({
        date: '2019-07-01',
        personA: 'Aさん',
        personB: 'Bさん',
        label: 'ペットボトルお茶',
        price: '150'
      });
      this.closeModal();
    },
    openAddModal: function() {
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
      const personA = this.$refs.personA_edit;
      const personB = this.$refs.personB_edit;
      const label = this.$refs.label_edit;
      const price = this.$refs.price_edit;

      if (!date.value.length || !personA.value.length || !personB.value.length || !label.value.length || !price.value.length) {
        this.inputError = true;
        return;
      }

      const datas = {
        date: date.value,
        personA: personA.value,
        personB: personB.value,
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
$color-main: #2c3e50;
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
  color: $color-main;
  margin: 0 auto;
  position: relative;
  padding: 50px 0;
}
.container {
  width: calc(100% - 40px);
  margin: 0 20px;
  position: relative;
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
    position: relative;
    &:after {
      content: '';
      position: absolute;
      right: 10px;
      top: 50%;
      transform: translateY(-45%);
      width: 0;
      height: 0;
      border-style: solid;
      border-width: 10px 8px 0 8px;
      border-color: #878787 transparent transparent transparent;
      z-index: -1;
    }
    select {
      cursor: pointer;
      border: 1px solid #bbb;
      padding: 3px 12px;
      padding-right: 32px;
      font-size: 16px;
      transform: scale(0.9);
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
  width: 100%;
  > .container {
    position: relative;
    button {
      position: absolute;
      bottom: 0;
      right: 10px;
      background-color: $color-main;
      width: 40px;
      height: 40px;
      border-radius: 50%;
      cursor: pointer;
      span {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        width: 60%;
        height: 3px;
        &:before, &:after {
          content: '';
          position: absolute;
          width: 100%;
          height: 100%;
          background-color: #fff;
        }
        &:before {
          transform: rotate(90deg);
        }
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
    padding: 40px 0;
    font-weight: bold;
    animation: fade both .5s;
    @keyframes fade {
      from {
        opacity: 0;
        transform: translateY(100px);
      }
      to {
        opacity: 1;
        transform: translateY(0px);
      }
    }
    &.confirmOpen::before {
      content: '';
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background-color: rgba($color: #000, $alpha: .7);
      z-index: 5;
    }
  }
  .close {
    cursor: pointer;
    position: absolute;
    top: -20px;
    left: 0;
    width: 20px;
    height: 20px;
    z-index: 4;
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
    font-weight: bold;
    h2 {
      font-size: 24px;
      text-align: center;
      margin-bottom: 20px;
    }
    .inputWrap {
      border-bottom: 1px solid #bbb;
      margin-bottom: 18px;
      > input {
        width: 100%;
        padding: 5px 0;
        font-size: 16px;
        font-weight: inherit;
        display: inline-block;
      }
      &.price {
        display: flex;
        align-items: baseline;
        font-size: 20px;
        // padding-bottom: 5px;
        margin-bottom: 25px;
        > input {
          font-size: inherit;
          text-align: right;
        }
      }
      &.person {
        display: flex;
        align-items: baseline;
        > input {
          width: 35%;
        }
        > span {
          margin: 0 auto;
          &.rent {
            margin-right: 0;
          }
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
    }
  }
  .confirmArea {
    position: absolute;
    bottom: 0px;
    z-index: 99;
    padding: 20px 0 30px;
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
    }
  }
}

.error {
  color: crimson;
  font-size: 14px;
  margin-bottom: 10px;
  text-align: center;
  font-weight: bold;
}
.c-btn {
  border-width: 1px;
  border-style: solid;
  font-size: 16px;
  padding: 6px 10px;
  font-weight: inherit;
  cursor: pointer;
  &-black {
    @extend .c-btn;
    background-color: $color-main;
    color: #fff;
    border-color: $color-main;
  }
  &-gray {
    @extend .c-btn;
    color: #999;
    border-color: #bbb;
  }
  &-red {
    @extend .c-btn;
    color: crimson;
    border-color: crimson;
  }
}

</style>
