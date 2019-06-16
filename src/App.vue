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
        <button class="submit">記録</button>
      </section>
    </div>

    <div class="app-content app-log">
      <h1>ログ画面</h1>
      <table>
        <tr>
          <th>2019-06-31</th>
          <td>もんち</td>
          <td>夕飯代</td>
          <td>1000円</td>
          <td><button>×</button></td>
        </tr>
        <tr>
          <th>2019-06-31</th>
          <td>もんち</td>
          <td>夕飯代</td>
          <td>1000円</td>
          <td><button>×</button></td>
        </tr>
      </table>
    </div>



  </div>
</template>

<script>
export default {
  name: 'app'
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
