<script setup lang="ts">
import { computed, ref, Ref } from "vue";
import { BPagination, BFormCheckbox } from "bootstrap-vue-3";
import ky from "ky";

interface MMPResult {
  count: number;
  next?: string;
  previous?: string;
  results: Array<any>;
}

let query: string = "";
const apiEndpoint = "https://mmp.acdh-dev.oeaw.ac.at/api/stelle/";
const queryField = ref("");
const page = ref(1);
const pageSize = ref(20);
const isLoading = ref(false);
const experimentalSorting = ref(false);

const data: Ref<MMPResult> = ref({ count: 0, results: [] });

const loadData = async () => {
  if (queryField.value === "") return;
  if (queryField.value !== query) {
    query = queryField.value;
    page.value = 1;
  }

  isLoading.value = true;
  const params = new URLSearchParams({
    zitat: query,
    zitat_lookup: "icontains",
    limit: pageSize.value.toString(),
    offset: ((page.value - 1) * pageSize.value).toString(),
  });

  try {
    data.value = await ky(apiEndpoint + "?" + params).json();
    console.debug(data.value);
  } catch (error) {
    alert("error fetching data: " + error);
  }
  if (data.value.count === 0) {
    alert("no results");
  }
  isLoading.value = false;
};

const loadPage = async (pageNum: number) => {
  page.value = pageNum;
  loadData();
};

const paginationUpdated = async () => {
  await loadData();
  window.scrollTo({ left: 0, top: 0, behavior: "smooth" });
};

const sortedResults = computed(() => {
  if (experimentalSorting.value) {
    const countQuery = (obj: any) => {
      var regEx = new RegExp(query, "ig");
      return (JSON.stringify(obj).match(regEx) || []).length;
    };
    return data.value.results
      .slice()
      .sort((a: any, b: any) => countQuery(b) - countQuery(a));
  }
  return data.value.results;
});

const highlightQuery = (text: string) => {
  var regEx = new RegExp(query, "ig");
  return text.replace(regEx, '<span class="highlight">$&</span>');
};

const getItemURI = (item: any) => {
  // I felt better extracting ID from url and not to use legacy_pk+1
  const id: string = item.url.match(/\d+/)[0];
  return "https://mmp.acdh-dev.oeaw.ac.at/archiv/stelle/detail/" + id;
};

const getKeywords = (item: any) =>
  item.key_word.map((e: any) => e.stichwort).join(", ");

const getAuthors = (item: any) =>
  item.text.autor.map((e: any) => e.name).join(", ");
</script>

<template>
  <article>
    <header class="entry-header">
      <h1 class="entry-title">MMP</h1>
    </header>
    <div class="entry-content">
      <div class="d-flex flex-md-row flex-column">
        <div class="form-group">
          <input
            class="form-control ds-input query-field"
            v-model="queryField"
            @keyup.enter="loadData()"
            placeholder="Suche..."
            spellcheck="false"
          />
        </div>
        <div class="form-group ml-md-4">
          <select
            v-model.number="pageSize"
            @change="loadPage(1)"
            class="form-control"
          >
            <option>5</option>
            <option>10</option>
            <option>20</option>
            <option>50</option>
            <option>100</option>
          </select>
        </div>
        <div class="form-group ml-md-5 pt-2">
          <b-form-checkbox v-model="experimentalSorting"
            >Experimentelles Sortieren</b-form-checkbox
          >
        </div>
        <div class="ml-auto pt-2">
          <span v-if="isLoading">Laden...</span>
          <span v-else-if="data.count > 0">Ergebnisse: {{ data.count }}</span>
        </div>
      </div>

      <table
        class="table table-striped table-responsive"
        v-if="data.count > 0"
        :class="{ loading: isLoading }"
      >
        <thead>
          <tr>
            <th>Label</th>
            <th>Stichworte</th>
            <th>Autor:innen</th>
            <th>Jahresspanne</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="item in sortedResults" v-bind:key="item">
            <td>
              <a :href="getItemURI(item)" target="_blank"
                ><span
                  class="highlighted-text"
                  v-html="highlightQuery(item.display_label)"
                ></span
              ></a>
            </td>
            <td>{{ getKeywords(item) }}</td>
            <td>{{ getAuthors(item) }}</td>
            <td>
              {{
                item.text.start_date +
                (item.text.start_date === "" || item.text.end_date === ""
                  ? ""
                  : " – ") +
                item.text.end_date
              }}
            </td>
          </tr>
        </tbody>
      </table>
      <b-pagination
        v-if="data.count / pageSize > 1"
        class="justify-content-center"
        v-model="page"
        :total-rows="data.count"
        :per-page="pageSize"
        :limit="7"
        :disabled="isLoading"
        @update:model-value="paginationUpdated()"
      ></b-pagination>
    </div>
  </article>
</template>

<style scoped>
.loading {
  opacity: 0.5;
}
a:hover {
  filter: none !important;
}
.highlighted-text :deep(.highlight) {
  background-color: rgb(242, 255, 156) !important;
}
</style>
