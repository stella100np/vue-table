<template>
  <div>
    <table border="1">
      <tr>
        <th v-for="(item,index) in columns" :key="index">
          {{item.title}}
          <button
            v-if="item.sortable === true"
            v-on:click="changeSortType(item.key, 'asc')"
          >升序</button>
          <button v-if="item.sortable === true" v-on:click="changeSortType(item.key, 'desc')">降序</button>
					<!-- <筛选> -->
          <template v-if="item.filters">
            <select name="筛选" v-model="filterItems[item.key]" @change="afterChange()" >
              <option
                v-for="(item,index) in item.filters"
                :value="item.value"
                :key="index"
              >{{item.label}}</option>
            </select>
											<!-- <充值筛选> -->
						<button v-on:click="resetFilter(item.key),afterChange()">重置</button>
          </template>
        </th>
      </tr>

      <tr v-for="(item,index) in pageTableData" :key="index">
        <td v-for="(column, index) in columns" :key="index">{{item[column.key]}}</td>
      </tr>
    </table>

    <!--分页-->
    <div class="page-bar">
      <ul>
        <li v-if="cur>1">
          <a v-on:click="cur--,afterChange()">上一页</a>
        </li>
        <li v-if="cur==1">
          <a class="banclick">上一页</a>
        </li>
        <li v-for="index in all" v-bind:class="{ 'active': cur == index}" :key="index">
          <a v-on:click="btnClick(index),afterChange()">{{ index }}</a>
        </li>
        <li v-if="cur!=all">
          <a v-on:click="cur++,afterChange()">下一页</a>
        </li>
        <li v-if="cur == all">
          <a class="banclick">下一页</a>
        </li>
        <li>
          <a>
            共
            <i>{{all}}</i>页
          </a>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: "Table",
  props: {
    columns: Array,
    initData: Array,
    filters: Array
  },
  data() {
    return {
      tableData: [],
      sortItems: {},
      filterItems: {},
      filterFunctions: {},
      index: 1,
      cur: 1,
      pageSize: 5,
      pageTableData: []
    };
  },
  mounted() {
    this.tableData = this.initData;
    this.setSortColums();
    this.setFilterColumns();
    this.initializePage();
  },
  computed: {
    all: function() {
      return Math.ceil(this.tableData.length / this.pageSize);
    }
  },
  methods: {
    initializePage() {
      this.pageTableData = this.tableData.slice(0, 5);
    },
    setSortColums() {
      let cols = this.columns.filter(col => "sortable" in col);
      for (let index = 0; index < cols.length; index++) {
        const element = cols[index];
        this.sortItems[element.key] = "normal";
      }
    },
    setFilterColumns() {
      let cols = this.columns.filter(col => "filters" in col);
      for (let index = 0; index < cols.length; index++) {
        const element = cols[index];
        this.filterItems[element.key] = "";
        this.filterFunctions[element.key] = element.filterMethod;
      }
    },
    btnClick(index) {
      this.cur = index;
    },
    changeSortType(colKey, type) {
      this.sortItems[colKey] = type;
      this.afterChange();
		},
		resetFilter(k){
			this.filterItems[k]=""
		},

    handleSort(data) {
      const obj = this.sortItems;
      let sortedResult = data;
      for (const key in obj) {
        const sortType = obj[key];
        if (sortType === "asc") {
          sortedResult = data.sort((a, b) => a[key] - b[key]);
        }
        if (sortType === "desc") {
          sortedResult = data.sort((a, b) => b[key] - a[key]);
        }
      }
      return sortedResult;
    },

    hanldeFilter(data) {
      let filterResult = data;
      for (const k in this.filterItems) {
        const v = this.filterItems[k];
        const func = this.filterFunctions[k];
        if (v) {
          filterResult = filterResult.filter(ele => func(v, ele));
        }
      }
      return filterResult;
    },

    handlePagination(data) {
      let pageData = data;
      if (pageData.length > 5) {
        return pageData.slice(
          (this.cur - 1) * this.pageSize,
          this.cur * this.pageSize
        );
      }
      return pageData;
    },

    afterChange() {
      this.tableData = this.initData;
      let res = this.hanldeFilter(this.initData);
      res = this.handleSort(res);
      this.tableData = res;
      this.pageTableData = this.handlePagination(res);
    }
  }
};
</script>
<style scoped>
/*分页*/
.page-bar {
  margin: 40px auto;
}
ul,
li {
  margin: 0px;
  padding: 0px;
}
li {
  list-style: none;
}
.page-bar li:first-child > a {
  margin-left: 0px;
}
.page-bar a {
  border: 1px solid #ddd;
  text-decoration: none;
  position: relative;
  float: left;
  padding: 6px 12px;
  margin-left: -1px;
  line-height: 1.42857143;
  color: #5d6062;
  cursor: pointer;
  margin-right: 20px;
}
.page-bar a:hover {
  background-color: #eee;
}
.page-bar a.banclick {
  cursor: not-allowed;
}
.page-bar .active a {
  color: #fff;
  cursor: default;
  background-color: #e96463;
  border-color: #e96463;
}
.page-bar i {
  font-style: normal;
  color: #d44950;
  margin: 0px 4px;
  font-size: 12px;
}
</style>