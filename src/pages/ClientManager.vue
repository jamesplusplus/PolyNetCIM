<template>
  <div>
    <my-breadcrumbs :item="navItems"></my-breadcrumbs>
    <div class="my-content">
      <div class="search-bar">
        <div class="search-item condition">
          <v-select
            size="1"
            :items="[$t('ByName'), $t('ByCode')]"
            :label="$t('QueryCondition')"
            v-model="nameOrCode"
          ></v-select>
        </div>
        <div class="search-item" v-if="nameOrCode === 'By Name' || nameOrCode === '通過姓名'">
          <v-text-field
            :label="$t('ByName')"
            v-model="params.cliLname"
          ></v-text-field>
        </div>
        <div class="search-item search_input"  v-if="nameOrCode === 'By Code' || nameOrCode === '通過賬號'">
          <v-text-field
            :label="$t('ByCode')"
            v-model="params.cliCode"
          ></v-text-field>
        </div>

        <div class="search-item">
          <v-select
            :items="countrys"
            item-text="label"
            item-value="value"
            :label="$t('Country')"
            v-model="params.cliCty"
          ></v-select>
        </div>

        <div class="search-item">
          <v-select
            :items="statusList"
            item-text="label"
            item-value="value"
            :label="$t('Status')"
            v-model="params.cliSt"
          ></v-select>
        </div>

        <div class="search-item">
          <v-select
            :items="types"
            item-text="label"
            item-value="value"
            :label="$t('ClientType')"
            v-model="params.cliType"
          ></v-select>
        </div>

        <div class="search-item">
          <v-btn color="info" icon @click="searchList" :loading="isLoading"><v-icon small>search</v-icon></v-btn>
        </div>
        <div class="search-item">
          <v-btn color="info" icon @click="resetList"><v-icon small>refresh</v-icon></v-btn>
        </div>
        <div class="search-item">
          <v-btn color="info" icon to="/client-manager-edit/add"><v-icon small>add</v-icon></v-btn>
        </div>
      </div>

      <v-data-table
        :headers="headers"
        :items="dataTable"
        hide-actions
        :loading="isLoading"
        class="data-tabel"
      >
        <template slot="items" slot-scope="props">
          <td class="text-xs-left"><v-btn flat icon color="info" :to="`/client-manager-edit/${props.item.cliCode}`"><v-icon small>edit</v-icon></v-btn></td>
          <td class="text-xs-left">{{ props.item.cliLname }}</td>
          <td class="text-xs-left">{{ props.item.cliCode }}</td>
          <td class="text-xs-left">{{ props.item.cliMainagt }}</td>
          <td class="text-xs-left">{{ props.item.cliAddr1 }}</td>
          <td class="text-xs-left">{{ props.item.cliCty }}</td>
          <td class="text-xs-left">{{ props.item.cliSrctype }}</td>
          <td class="text-xs-left">{{ props.item.tmp_typeDesc }}</td>
          <td class="text-xs-left">{{ props.item.cliTelidd + props.item.cliTel }}</td>
          <td class="text-xs-left">{{ props.item.cliTelidd + props.item.cliFax }}</td>
        </template>
      </v-data-table>
      <div class="text-xs-center">
        <v-pagination
          :total-visible="7"
          v-model="params.page.currentPageNo"
          :length="pageLangth"
          :disabled="isLoading"
          @next="jump('next')"
          @previous="jump('prev')"
          @input="jump"
          class="pagination"
        ></v-pagination>
        <input type="text" v-model="currentNum" class="current_input" :placeholder="$t('currentNum')">
        <v-btn class="goJump" small @click="jump(currentNum)">Go</v-btn>
      </div>
    </div>
  </div>
</template>
<script>
// import myUpload from '../components/uploadFile.vue'
export default {
  name: 'ClientManager',
  data () {
    return {
      file: '',
      isLoading: false,
      pageLangth: 0,
      navItems: [
        {
          text: this.$t('ClientManager'),
          disabled: true,
          href: '/client-manager'
        }
      ],
      nameOrCode: 'By Name',
      params: {
        cliLname: '',
        cliCode: '',
        cliCty: '',
        cliType: '',
        cliSt: '',
        page: {
          currentPageNo: 1
        }
      },
      currentNum: null,
      countrys: [
        {
          value: 'MAC',
          label: 'MACAU'
        },
        {
          value: 'JAP',
          label: 'JAPAN'
        },
        {
          value: 'KOR',
          label: 'KOREA(SOUTH)'
        },
        {
          value: 'MAL',
          label: 'MALAYSIA'
        },
        {
          value: 'MAU',
          label: 'MAURITIUS'
        },
        {
          value: 'MEX',
          label: 'MEXICO'
        }
      ],
      statusList: [
        {
          value: 'A',
          label: 'Active'
        },
        {
          value: 'C',
          label: 'Close'
        },
        {
          value: 'D',
          label: 'Delete'
        },
        {
          value: 'S',
          label: 'Suspend'
        }
      ],
      types: [],
      headers: [
        {
          text: '',
          sortable: false
        },
        {
          text: this.$t('LongName'),
          value: 'cliLname'
        },
        {
          text: this.$t('Client'),
          value: 'cliCode'
        },
        {
          text: this.$t('MainAgt'),
          sortable: false
        },
        {
          text: this.$t('Address'),
          value: 'cliAddr1'
        },
        {
          text: this.$t('Country'),
          value: 'cliCty'
        },
        {
          text: this.$t('Type'),
          sortable: false
        },
        {
          text: this.$t('TypeDesc'),
          sortable: false
        },
        {
          text: this.$t('TelNo.'),
          value: 'cliTelidd' + 'cliTel'
        },
        {
          text: this.$t('FaxNo.'),
          value: 'cliTelidd'
        }
      ],
      dataTable: []
    }
  },
  mounted () {
    this.queryClientType()
    this.queryList()
  },
  methods: {
    /**
     * 获取client type 下拉列表
     */
    queryClientType () {
      this.$fetch({
        method: 'POST',
        url: '/ipoly/clientManager/getClientType.json'
      })
      .then(res => {
        let types = []
        res.body.map(item => {
          let obj = {}
          obj.label = item.cleName
          obj.value = item.cleCode
          types.push(obj)
        })
        this.types = types
      })
    },
    /**
     * 获取页面表单数据
     */
    queryList () {
      this.isLoading = true
      this.$fetch({
        method: 'POST',
        url: '/ipoly/clientManager/getList.json',
        body: this.params
      })
      .then(res => {
        this.dataTable = res.body.data
        this.pageLangth = res.body.page.totalPageNum
        this.isLoading = false
      })
    },
    /**
     * 根据参数搜索列表
     */
    searchList () {
      this.params.page.currentPageNo = 1
      this.queryList()
    },
    /**
     * 重置表单
     */
    resetList () {
      this.params.cliLname = ''
      this.params.cliCode = ''
      this.params.cliCty = ''
      this.params.cliSt = ''
      this.params.cliType = ''
    },
    /**
     * 分页功能回调函数
     * @param {number,string} type next=下一页 prev=上一页 Number类型=跳到当前页
     */
    jump (type) {
      if (type === 'next') {
        this.params.page.currentPageNo + 1
      } else if (type === 'prev') {
        this.params.page.currentPageNo - 1
      } else {
        this.params.page.currentPageNo = parseInt(type)
      }
      this.queryList()
    }
  }
}
</script>
<style lang="less">
.data-icon {
  float: left;
  margin-right: 5px;
}
.search-bar {
  padding: 10px 28px 10px 28px;
}
.search-bar .search-item {
  max-width: 100%;
}
.search-bar .search-item:nth-child(2) {
  width: 280px !important;
}
.my-content {
  background: #f2f8fb;
}
.data-tabel tr:hover td {
  background-color: #ceedfe;
}
.data-tabel tr:nth-child(even) td {
  background-color: #f7f7f7 !important;
}
.data-tabel tr:nth-child(even):hover td {
  background-color: #ceedfe !important;
}
.theme--light.v-table tbody tr:not(:last-child) {
  border: none;
}
table.v-table thead th {
  font-size: 14px;
  color: #000;
  font-weight: bolder;
}
.v-btn--icon {
  border-radius: 10%;
}
.pagination {
  vertical-align: middle;
}
.current_input {
  width: 60px;
  height: 28px;
  font-size: 12px;
  text-align: center;
  border: 1px solid rgba(177,176,176,1);
  border-radius: 5%;
  outline: none;
  color: #000;
  margin-right: 10px;
  position: relative;
  top: 2px;
}

</style>
