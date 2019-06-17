<template>
  <div id="app">
    <div class="app-content app-input">
      <h1>入力画面</h1>
      <section>
        <h2>いつ</h2>
        <input type="date" name="input1">
      </section>
      <section>
        <h2>誰が</h2>
        <input type="text" name="input2">
      </section>
      <section>
        <h2>何に</h2>
        <input type="text" name="input3">
      </section>
      <section>
        <h2>いくら払った</h2>
        <input type="number" name="input4">円
      </section>
      <section>
        <button class="submit" v-on:click="createLog()">記録</button>
      </section>
    </div>

    <div class="app-content app-log">
      <h1>ログ画面</h1>
      <table>
        <tr v-for="(mochi, key) in mochiRef">
          <th>{{mochi.date}}</th>
          <td>{{mochi.person}}</td>
          <td>{{mochi.what}}</td>
          <td>{{mochi.yen}}</td>
          <td><button>×</button></td>
        </tr>
      </table>
    </div>



  </div>
</template>

<script>
export default {
  name: 'app',
  data () {
    return {
      database: null,
      mochiRef: null,
      newMochiLog: '',
      mochiLogs: [],
      items: [
        {
          date: 1,
          person: 'もんち',
          what: 'label',
          yen: 2000
        }
      ]
    }
  },
  created: function() {
    this.database = firebase.database();
    this.mochiRef = this.database.ref('mochi-log');

    var _this = this;
    this.mochiRef.on('value', function(snapshot) {
      _this.mochiLogs = snapshot.val();
    });
  },
  methods: {
    // createLog: function() {
    //   if (this.newMochiLog == "") { return; }
    //   this.mochiRef.push({
    //     name: this.newMochiLog
    //   })
    //   this.newTodoName = "";
    // }
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
