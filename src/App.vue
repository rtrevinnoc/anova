<template>
  <div class="w-screen max-w-sm text-center bg-light border border-dark rounded rounded-sm" style="padding: 15px; margin: 0 auto">
    <table class="w-full">
      <tr>
        <th>f(x<sub>i</sub>)</th>
      </tr>
      <tr v-for="i in (n_data + 1)" v-bind:key="i">
        <td><input v-for="j in (n_trat + 1)" v-bind:key="j" type="text" name="fs{{(i-1)}}{{(j-1)}}" id="fs{{(i-1)}}{{(j-1)}}" v-model.number="fs[(i-1)][(j-1)]"></td>
      </tr>
    </table>
    <br>
    <table>
      <tr>
          <td>Source of Variation</td>
          <td>Sum of Squares</td>
          <td>df</td>
          <td>Mean Square</td>
          <td>F</td>
      </tr>
          <td>Regression (explained)</td>
          <td>2920.68100</td>
          <td>{{ this.dfb }}</td>
          <td>2920.68100</td>
          <td>{{ this.f_calc }}</td>
      <tr>
          <td>Residual (unexplained)</td>
          <td>4342.38800</td>
          <td>{{ this.dfw }}</td>
          <td>542.79850</td>
      </tr>
      <tr>
          <td>Total</td>
          <td>{{ this.SSt }}</td>
          <td>{{ this.dft }}</td>
      </tr>
    </table>
    <br>
    <button v-on:click="increase_data()">Agregar dato</button>
    <button v-on:click="decrease_data()">Eliminar dato</button>
    <button v-on:click="increase_trat()">Agregar tratamiento</button>
    <button v-on:click="decrease_trat()">Eliminar tratamiento</button>
    <button v-on:click="solve()">Calcular</button>
  </div>
</template>

<script>
// import fdistr from '@/statistics-distributions'
// import { create, all } from "mathjs";

// const math = create(all, {});

Number.prototype.round = function(n) {
  const d = Math.pow(10, n);
  return Math.round((this + Number.EPSILON) * d) / d;
}

export default {
  name: "App",
  components: {},
  data() {
    return {
      n_data: 2,
      n_trat: 2,
      n: 1,
      avgs: [],
      avg: 0,
      SSt: 0,
      SSw: 0,
      SSb: 0,
      Sqb: 0,
      Sqw: 0,
      fs: [[2,2,1], [4,3,2], [2,4,5]],
      dfb: 0,
      dfw: 0,
      dft: 0,
      f_table: 0,
      f_calc: 0,
      significance: 0.05,
    };
  },
  methods: {
    increase_data() {
      this.n_data += 1;
      this.fs.push(Array(this.n_trat + 1).fill(null));
    },
    decrease_data() {
      this.n_data -= 1;
      this.fs.pop();
    },
    increase_trat() {
      this.n_trat += 1;
      this.fs.forEach((list) => {
        list.push(null);
      });
    },
    decrease_trat() {
      this.n_trat -= 1;
      this.fs.forEach((list) => {
        list.pop();
      });
    },
    solve() {
      this.n = (this.n_data + 1) * (this.n_trat + 1);
      this.dfb = (this.n_data + 1) - 1;
      this.dfw = this.n - (this.n_data + 1);
      this.dft = this.dfb + this.dfw;
      this.avgs = this.fs[0].map((col, i) => (this.fs.map(row => row[i]).reduce((acc, c) => acc + c, 0) / this.fs.length).round(3));
      this.avg = (this.avgs.reduce((acc, x) => acc + x) / this.avgs.length).round(3)
      this.SSt = this.fs[0].map((col, i) => this.fs.map(row => row[i]).reduce((acc, c) => acc + ((c - this.avg) ** 2), 0)).reduce((acc, y) => acc + y).round(3);
      this.SSw = this.fs[0].map((col, i) => this.fs.map(row => row[i]).reduce((acc, c) => acc + ((c - this.avgs[i]) ** 2), 0)).reduce((acc, y) => acc + y).round(3);
      this.SSb = this.SSt - this.SSw
      this.Sqb = this.SSb/this.dfb
      this.Sqw = this.SSw/this.dfw
      this.f_calc = (this.Sqb/this.Sqw).round(3)
    }
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin: 60px;
}
</style>
