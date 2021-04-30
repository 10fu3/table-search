<template>
  <v-row justify="center" align="center">
    <div class="text-center">
      <p
        class="text-h3 font-weight-light ml-5 mr-5"
        style="color: #707070; margin-top: 200px"
      >
        非公式 芝浦工大 授業情報
      </p>

      <div>
        <v-combobox
          @update:search-input="suggest"
          label="授業名を検索"
          :items="this.hit"
          prepend-inner-icon="mdi-magnify"
          single-line
          color="green"
          style="margin-left: 30px; margin-right: 30px"
          class="mt-12"
          rounded
          outlined
          @keydown.enter="search"
        ></v-combobox>
        <v-btn
          x-large
          style=""
          class="text-h5"
          outlined
          color="#606060"
          @click="search"
        >
          <p class="text-h5" style="padding-top: 12px">検索</p>
        </v-btn>
      </div>
    </div>
  </v-row>
</template>

<script>
export default {
  data() {
    return {
      store: {
        exist: false,
      },
      hit: [],
      word: "",
    };
  },
  created: function () {
    if (
      localStorage.getItem("syllabus") === null ||
      localStorage.getItem("timetable") === null
    ) {
      this.store.exist = false;
      return;
    } else {
      if (
        this.getTimetable() === undefined ||
        this.getTimetable().length === 0
      ) {
        this.$store.commit(
          "timetable/setData",
          JSON.parse(localStorage.getItem("timetable"))
        );
      }
      if (
        this.getSyllabus() === undefined || this.getSyllabus() === null || this.getSyllabus() === {} || Object.keys(this.getSyllabus()).keys.length == 0
      ) {
        this.$store.commit(
          "syllabus/setData",
          JSON.parse(localStorage.getItem("syllabus"))
        );
      }
      this.store.exist = true;
    }
  },
  methods: {
    getSyllabus: function () {
      return this.$store.state.syllabus.data;
    },
    getTimetable: function () {
      return this.$store.state.timetable.data;
    },
    suggest: function (e) {
      if(e === undefined || e === null || e.length === 0){
        this.hit = [];
      }else{
        const datas = this.getSyllabus();
        this.hit = datas.flat().filter(d=>d.name.includes(e)).map(d=>d.name+' '+(d.division === undefined ? '学部:'+d.big_category : '学科:'+d.division));
        this.word = e;
      }
    },
    search: function () {
      if (this.word && this.word.length > 0) {
        this.$router.push("/result?search=" + this.word);
      }
    },
  }
}
</script>