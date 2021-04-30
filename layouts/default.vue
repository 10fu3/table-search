<template>
  <v-app dark>
    <v-app-bar :clipped-left="clipped" fixed app color="green">
      <v-toolbar-title>
        <div style="color: white">{{ title }}</div>
      </v-toolbar-title>
      <v-spacer />
    </v-app-bar>
    <v-main>
      <v-container>
        <v-dialog v-model="!store.exist" persistent width="500">
          <v-card>
            <v-card-title class="headline grey lighten-2">
              ダウンロードする学部を選択
            </v-card-title>

            <v-card-text>
              <v-expansion-panels>
                <v-expansion-panel v-for="(bigDivision, i) in Object.keys(list)" :key="i">
                  <v-expansion-panel-header>
                    {{bigDivision}}
                  </v-expansion-panel-header>
                  <v-expansion-panel-content>
                    <v-list>
                      <v-list-item
                        v-for="(smallDivision, j) in list[bigDivision]"
                        v-bind:key="i - j"
                      >
                        <v-list-item-title>
                          {{ smallDivision.display }}
                        </v-list-item-title>
                        <v-list-item-action>
                          <v-checkbox v-model="list[bigDivision][j]['flag']"> </v-checkbox>
                        </v-list-item-action>
                      </v-list-item>
                    </v-list>
                  </v-expansion-panel-content>
                </v-expansion-panel>
              </v-expansion-panels>
            </v-card-text>
            <v-divider></v-divider>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="primary" outlined text @click="downloadData">
                ダウンロードする
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <nuxt />
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      downloadTarget: [],
      list: {},
      store: {
        exist: false,
      },
    };
  },
  created: async function () {
    this.list = await this.getDivisionList();
  },
  mounted: function () {
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
      if (this.getSyllabus() === undefined) {
        this.$store.commit(
          "syllabus/setData",
          JSON.parse(localStorage.getItem("syllabus"))
        );
      }

      this.store.exist = true;
    }
  },
  methods: {
    puts: function (o) {
      console.log(JSON.parse(JSON.stringify(o)));
    },
    getSyllabus: function () {
      return this.$store.state.syllabus.data;
    },
    getTimetable: function () {
      return this.$store.state.timetable.data;
    },
    getJSONArray: async function (urls) {
      let results = [];
      let resultJSONs = [];
      for (let url of urls) {
        results.push(fetch(url));
      }
      let getdata = (await Promise.all(results));

      for(let r of getdata){
        resultJSONs.push({
          url: r.url,
          body: (await r.json()),
        });
      }
      return resultJSONs;
    },

    getDivisionList: async function () {
      const data = await (
        await fetch("https://10fu3.github.io/shibaura-timetable-api/list.json")
      ).json();
      const result = {};
      for(const t of Object.keys(data)){
        result[t] = data[t].map((s) => {
          let a = {};
          a.big = t;
          a.name = s;
          a.display = s.replaceAll("_", " ");
          a.flag = false;
          return a;
        });
      }
      return result;
    },

    downloadData: async function () {
      const targets = Object.keys(this.list).map(d=>this.list[d]).flat().filter(d=>d.flag);

      var syllabusURLs = targets.map(d=>'https://10fu3.github.io/shibaura-timetable-api/syllabus/'+d.name+'.json');

      const tableList = (await (await fetch("https://10fu3.github.io/shibaura-timetable-api/timetable/list.json")).json());

      const table = tableList.filter(d=>syllabusURLs.filter(u=>u.includes(d)).length > 0).map(u=>'https://10fu3.github.io/shibaura-timetable-api/timetable/'+u+'.json');

      //時間割データ
      const tableDatas = (await this.getJSONArray(table)).map(d=>d.body);

      //storeに保存
      this.$store.commit("timetable/setData",tableDatas);

      //localStorageに保存
      localStorage.setItem("timetable",JSON.stringify(this.getTimetable()));

      if(targets.filter(d=>d.big === 'システム理工学部').length > 0){
        syllabusURLs.push('https://10fu3.github.io/shibaura-timetable-api/syllabus/学科共通.json');
      }

      //シラバスデータ
      const syllabusDatas = (await this.getJSONArray(syllabusURLs)).map(d=>d.body);

      //storeに保存
      this.$store.commit("syllabus/setData", syllabusDatas);
      //localStorageに保存
      localStorage.setItem("syllabus",JSON.stringify(this.getSyllabus()));

      this.store.exist = true;
    },
  },
};
</script>