<template>
  <div class="bible-reader">
    <el-row v-if="language=='cn'">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item :to="{ name: 'home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item :to="{ name: 'bibleIndex' , params: {version: 1}, query: {language: language}}">目录</el-breadcrumb-item>
        <el-breadcrumb-item>{{scripture.name_cn}}</el-breadcrumb-item>
        <el-breadcrumb-item>第{{chapter}}章</el-breadcrumb-item>
      </el-breadcrumb>
      <p>{{scripture.name_cn}}</p>
    </el-row>
    <el-row v-else>
      <el-breadcrumb separator="/">
        <el-breadcrumb-item :to="{ name: 'home' }">Home</el-breadcrumb-item>
        <el-breadcrumb-item :to="{ name: 'bibleIndex', params: {version: 2}, query: {language: language} }">Index</el-breadcrumb-item>
        <el-breadcrumb-item>{{scripture.name_en}}</el-breadcrumb-item>
        <el-breadcrumb-item>Chapter {{chapter}}</el-breadcrumb-item>
      </el-breadcrumb>
      <p>{{scripture.name_en}}</p>
    </el-row>
    <el-row :gutter="6">
      <el-col :span="2" v-for="n in scripture.chapters" :key="scripture.abbr_en + '-' + n"><div class="grid-content ch-item"  @click="showVerses({version: version, scripture: scriptureId, chapter: n})"  :class="{active: isActive(n,chapter)}">{{n}}</div></el-col>
    </el-row>
    <el-row>
      <p v-for="(verse,index) in verses" :key="'verse-' + index">{{verse.verse}}  {{verse.content}}</p>
    </el-row>
  </div>
</template>

<script>
  export default {
    name: 'bibleReader',
    data() {
      return {
        scripture: {
          name_cn: '',
          name_en: '',
          abbr_en: '',
          chapters: 0,
        },
        version: 1,
        scriptureId: 1,
        chapter: 1,
        abbr_en: 'Gen',
        verses: [],
        language: 'cn',
      };
    },
    created() {
      this.chapter = this.$route.query.chapter || 1;
      this.abbr_en = this.$route.params.abbr_en || 'Gen';
      this.version = this.$route.params.version || 1;
      this.scriptureId = this.$route.params.scripture || 1;
      this.chapter = this.$route.params.chapter || 1;
      this.language = this.$route.query.language || 'cn';
      this.scripture = this.$store.getters.scripture({
        version: this.version,
        scripture: this.scriptureId,
      });

      this.getVerses({
        version: this.version,
        scripture: this.scriptureId,
        chapter: this.chapter,
      });

      this.$electron.ipcRenderer.on('chapter-read-reply', (event, data) => {
        this.verses = data || [];
      });
    },
    methods: {
      getVerses(arg) {
        this.$electron.ipcRenderer.send('chapter-read', arg);
      },
      showVerses(arg) {
        this.$router.push({ name: 'reader', params: arg });
        this.getVerses(arg);
      },
      isActive(chapter) {
        this.chapter = this.$route.params.chapter || 1;
        return this.chapter === chapter;
      },
    },
    destroyed() {
      this.$electron.ipcRenderer.removeAllListeners('chapter-read-reply');
    },
    beforeRouteUpdate(to, from, next) {
      if (to.fullPath !== from.fullPath) {
        this.getVerses(to.params);
      }
      if (to.params.scripture !== from.params.scripture) {
        this.scripture = this.$store.getters.scripture({
          version: to.params.version,
          scripture: to.params.scripture,
        });
      }
      next();
    },
  };
</script>

<style>
.ch-item {
  border: solid #99CCCC 1px;
  margin: 3px;
  padding: 3px;
  text-align: center;
  cursor: pointer;
}

.ch-item:hover {
  border-color: red;
}

.grid-content.active {
  background: #67C23A;
}
</style>