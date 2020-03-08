<template>
  <div>
    <div
      v-for="(v, k) in indices"
      :key="v.uuid"
      @click="curIndex = k"
    >{{ k }}, {{ v.total.docs.count }} documents, {{ Number.parseFloat(v.total.store.size_in_bytes/(1024*1024)).toPrecision(2) }} MiB</div>
    <hr />
    {{ mapping }}
    <hr />
    <p v-for="d in docs.hits.hits" :key="d._id">
      {{ d._type }}
      <span
        v-for="(value, name, idx) in d._source"
        :key="d._id + idx"
      >{{ name }} ({{ mapping[d._type].properties[name].type }}) {{ value }}</span>
    </p>
  </div>
</template>

<script>
import axios from "axios";

export default {
  created() {
    axios
      .get("http://localhost:9200/_stats/docs,store")
      .then(r => (this.indices = r.data.indices));
    this.refreshData();
  },
  data() {
    return {
      curIndex: "quantcopy",
      indices: [],
      mapping: [],
      docs: { hits: { hits: [] } }
    };
  },
  watch: {
    curIndex() {
      this.refreshData();
    }
  },
  methods: {
    refreshData() {
      axios
        .get(`http://localhost:9200/${this.curIndex}/_mapping`)
        .then(r => (this.mapping = r.data[this.curIndex].mappings));
      axios
        .get(`http://localhost:9200/${this.curIndex}/_search`, {
          query: { match_all: {} }
        })
        .then(r => (this.docs = r.data));
    }
  }
};
</script>