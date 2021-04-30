<template>
  <div class="">
    <v-btn
      @click="$router.go(-1)"
      rounded
      outlined
      style="margin-left: 30px"
      class="mt-12"
    >
      戻る
    </v-btn>
    <v-autocomplete
      label="授業名を検索"
      prepend-inner-icon="mdi-magnify"
      hide-no-data
      single-line
      color="green"
      style="margin-left: 30px; width: 40%; min-width: 350px"
      class="mt-12"
      :search-input.sync="search"
      :items="hit"
      @keydown.enter="updateData(search)"
      @update:search-input="suggest"
      rounded
      outlined
    ></v-autocomplete>

    <p class="text-h3 ml-8 font-weight-light" style="color: #707070">
      {{ this.$route.query.search }}の検索結果
    </p>

    <v-row class="mr-10 ml-10">
      <v-list>
        <v-list-item
          v-for="(time, index) in result"
          v-bind:key="'time.' + index"
        >
          <v-list-item-content>
            <v-card class="mt-5">
              <v-card-title @click="onClick(index)">
                <p class="text-h3 font-weight-light" style="color: #707070">
                  {{ time.name }}
                </p>
              </v-card-title>
              <v-card-text>
                <p
                  class="text-h5 font-weight-light5"
                  style="color: #707070"
                  @click="onClick(index)"
                >
                  担当 {{ time.teachers.join(" ") }}
                </p>
                <v-row class="mb-3" style="padding-left: 10px">
                  <div
                    v-for="(tag, tagindex) in Object.keys(time)"
                    v-bind:key="'time.' + index + '.' + tagindex"
                  >
                    <v-btn
                      v-if="time[tag] === '必修'"
                      outlined
                      @click="onClickTag(descriptions[tag] + ':' + time[tag])"
                      rounded
                      style="color: red"
                      class="mt-3 mr-1 ml-1"
                    >
                      {{ descriptions[tag] + ":必修" }}
                    </v-btn>
                    <v-btn
                      v-else-if="
                        tag !== 'teachers' &&
                        Object.keys(descriptions).includes(tag)
                      "
                      @click="onClickTag(descriptions[tag] + ':' + time[tag])"
                      outlined
                      rounded
                      class="mt-3 mr-1 ml-1"
                    >
                      {{
                        descriptions[tag] +
                        ":" +
                        (String(time[tag]).split("　").length > 2
                          ? String(time[tag]).split("　")[0] +
                            " " +
                            String(time[tag]).split("　")[1]
                          : String(time[tag]).split("　")[0])
                      }}
                    </v-btn>
                  </div>
                </v-row>
              </v-card-text>
            </v-card>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-row>
  </div>
</template>

<style>
.container {
  max-width: 100vw;
  padding: 0px;
}
</style>

<script>
export default {
  created: function () {
    this.search = this.$route.query.search;

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
        this.getSyllabus() === undefined ||
        this.getSyllabus() === null ||
        this.getSyllabus() === {} ||
        Object.keys(this.getSyllabus()).keys.length == 0
      ) {
        this.$store.commit(
          "syllabus/setData",
          JSON.parse(localStorage.getItem("syllabus"))
        );
      }
    }
  },
  methods: {
    joinWords: function (input) {
      if (input === null) {
        return [];
      }
      let words = input.split(" ");
      var temp = "";
      const result = [];

      for (const word of words) {
        if (word.includes(":")) {
          result.push(temp);
          result.push(word);
          temp = "";
        } else {
          if (temp.length !== 0) {
            temp += " ";
          }
          temp += word;
        }
      }
      return result;
    },
    updateData: async function (search) {
      
      if (search === null || search.length === 0) {
        return;
      }

      var words = this.joinWords(search);
      if(words.length === 0){
          words = [search];
      }

      const data = this.getSyllabus().flat();

      var r = [];
      for (const tag of words.filter((d) => !d.includes(":"))) {
        data.filter((d) => {
            return d.name.includes(tag);
        }).forEach((e) => {
            r.push(e);
        });
      }

      if (words.filter((d) => d.includes(":")).length > 0) {
        for (const tag of words.filter((d) => d.includes(":"))) {
          const t = tag.split(":");
          if(t.length === 1){
              break;
          }
          for (const f of Object.keys(this.descriptions)) {
            if (this.descriptions[f] === t[0]) {
              r = r.filter((d) => {
                  return  (d[f] !== undefined ? d[f] : "").includes(t[1]);
              });
            }
          }
        }
      }

      this.result = r;
    },
    getSyllabus: function () {
      return this.$store.state.syllabus.data;
    },
    getTimetable: function () {
      return this.$store.state.timetable.data;
    },
    suggest: function (e) {
      if (e === undefined || e === null || e.length === 0) {
        this.hit = [];
      } else {
        const datas = this.getSyllabus();
        this.hit = datas
          .flat()
          .filter((d) => d.name.includes(e))
          .map(
            (d) =>
              d.name +
              " " +
              (d.division === undefined
                ? "学部:" + d.big_category
                : "学科:" + d.division)
          );
        this.word = e;
      }
    },
    onClick: function (index) {
      //   this.$router.push("/detail?id=" + this.result[index].id);
    },
    onClickTag: function (tag) {},
    onEnter: function (search) {
      this.$router.push("/result?search=" + this.search);
    },
  },
  watch: {
    search: function (val, oldVal) {
      if (val === null) {
        val = this.$route.query.search;
      }
      this.$route.query.search = val;
    },
  },
  data() {
    return {
      search: "",
      descriptions: {
        type: "種別",
        credit: "単位数",
        class_count: "授業数",
        semester: "学期",
        big_category: "学部",
        division: "学科",
        grade: "学年",
      },
      result: [],
      hit: [],
    };
  },
};
</script>