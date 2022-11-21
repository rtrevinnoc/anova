<template>
  <div class="w-screen max-w-sm text-center bg-light border border-dark rounded rounded-sm" style="padding: 15px; margin: 0 auto">
    <table class="w-full">
      <tr>
        <th>f(x<sub>i</sub>)</th>
      </tr>
      <tr v-for="i in (n_data + 1)" v-bind:key="i">
        <td><input v-for="j in (n_trat + 1)" v-bind:key="j" type="number" name="fs{{(i-1)}}{{(j-1)}}" id="fs{{(i-1)}}{{(j-1)}}" v-model.number="fs[(i-1)][(j-1)]"></td>
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
        <td>P</td>
      </tr>
        <td>Regression (explained)</td>
        <td>{{ this.SSb }}</td>
        <td>{{ this.dfb }}</td>
        <td>{{ this.Sqb }}</td>
        <td>{{ this.f_calc }}</td>
        <td>{{ this.p_value }}</td>
      <tr>
        <td>Residual (unexplained)</td>
        <td>{{ this.SSw }}</td>
        <td>{{ this.dfw }}</td>
        <td>{{ this.Sqw }}</td>
      </tr>
      <tr>
        <td>Total</td>
        <td>{{ this.SSt }}</td>
        <td>{{ this.dft }}</td>
      </tr>
    </table>
    <br>
    &alpha; = <input type="number" name="significance" id="significance" placeholder="significance" v-model.number="significance">
    <br>
    <button v-on:click="increase_data()">Agregar dato</button>
    <button v-on:click="decrease_data()">Eliminar dato</button>
    <button v-on:click="increase_trat()">Agregar tratamiento</button>
    <button v-on:click="decrease_trat()">Eliminar tratamiento</button>
    <button v-on:click="solve()">Calcular</button>
    <br>
    <p>Resultado: {{ this.result }}</p>
  </div>
</template>

<script>
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
      p_value: 0,
      f_calc: 0,
      significance: 0.05,
      result: ""
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
    fhelper(t,r,a,e) {
      for (var n = 1, s = n, d = r; a >= d;) n = n * t * d / (d - e), s += n, d += 2;
      return s
    },
    fdist(t,r,a) {
      var e = a / (r * t + a);
      if (r % 2 == 0) return this.fhelper(1 - e, a, r + a - 4, a - 2) * Math.pow(e, a / 2);
      if (a % 2 == 0) return 1 - this.fhelper(e, r, r + a - 4, r - 2) * Math.pow(1 - e, r / 2);
      var n = Math.atan(Math.sqrt(r * t / a)),
          s = n / (Math.PI / 2),
          d = Math.cos(n),
          o = Math.sin(n);
      if (a > 1 && (s += d * o * this.fhelper(d * d, 2, a - 3, -1) / (Math.PI / 2)), 1 == r) return 1 - s;
      var l = 4 * this.fhelper(o * o, a + 1, r + a - 4, a - 2) * o * Math.pow(d, a) / Math.PI;
      if (1 == a) return 1 - s + l / 2;
      for (var h = 2;
          (a - 1) / 2 >= h;) l = l * h / (h - .5), h += 1;
      return 1 - s + l
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
      this.SSb = (this.SSt - this.SSw).round(3)
      this.Sqb = (this.SSb/this.dfb).round(3)
      this.Sqw = (this.SSw/this.dfw).round(3)
      this.f_calc = (this.Sqb/this.Sqw).round(3)
      this.p_value = this.fdist(this.f_calc, this.dfb, this.dfw).round(3)

      console.log(this.SSb, this.SSw, this.SSt);
      console.log(this.dfb, this.dfw, this.dft);
      console.log(this.p_value)

      if (this.p_value > this.significance) {
        this.result = "Se acepta la hipotesis nula, no hay diferencia estadisticamente signficativa entre las medias."
      } else {
        this.result = "Se rechaza la hipotesis nula, al menos una de las medias es distinta al resto."
      }
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

th, td {
  border: 1px solid white;
  border-collapse: collapse;
}
th {
  background-color: #F1A380;
}

td {
  background-color: #96D4D4;
}
</style>
