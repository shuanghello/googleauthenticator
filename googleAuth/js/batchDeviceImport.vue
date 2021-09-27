<template>
  <div class="flex-column">
    <div class="filter-container">
      <el-row :gutter="4" style="background-color: #FFFFFF; padding: 20px 10px;">
        <el-form ref="searchForm" :model="listQuery" size="mini" label-width="80px">
          <el-col :span="4">
            <el-form-item label="设备名称" prop="deviceName">
              <el-input v-model="listQuery.deviceName" placeholder="请输入设备名称" clearable :style="{width: '100%'}">
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="4">
            <el-form-item label="规格型号" prop="deviceModel">
              <el-input v-model="listQuery.deviceModel" placeholder="请输入规格型号" clearable :style="{width: '100%'}">
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="4">
            <el-form-item label="设备类型" prop="deviceType">
              <kl-select v-model="listQuery.deviceType" placeholder="请选择设备类型" clearable
                         :style="{width: '100%'}"
                         controller="Categorys"
                         method="LoadSelectDataSourceByType"
                         typeId="EQUIPMENT_TYPE"
                         @selectChange="deviceTypeChange"
              ></kl-select>
            </el-form-item>
          </el-col>
          <el-col :span="4">
            <el-form-item label="使用状态" prop="deviceStatus">
              <el-select v-model="listQuery.deviceStatus" placeholder="请选择设备状态" clearable
                         :style="{width: '100%'}">
                <el-option v-for="(item, index) in DeviceStatusOptions" :key="index" :label="item.label"
                           :value="item.value" :disabled="item.disabled"></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="4">
            <el-form-item label="位号" prop="locationCode">
              <el-input v-model="listQuery.locationCode" placeholder="请输入位号" clearable :style="{width: '100%'}">
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="4">
            <el-form-item label="存放位置" prop="location">
              <el-input v-model="listQuery.location" placeholder="请输入存放位置" clearable :style="{width: '100%'}">
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="4">
            <el-form-item label="所属单位" prop="department">
              <kl-cascader v-model="listQuery.department" :props="DepartmentProps"
                           :style="{width: '100%'}" :show-all-levels="false"
                           filterable placeholder="请选择所属单位"
                           controller="Orgs"
                           methods="GetTreeData"
                           @change="departmentChange"
              ></kl-cascader>
            </el-form-item>
          </el-col>
          <el-col :span="4">
            <el-form-item>
              <el-button type="primary" icon="el-icon-search" size="mini" @click="handleFilter(false)">搜索</el-button>
              <el-button type="warning" icon="el-icon-delete" size="mini" @click="handleFilter(true)">重置</el-button>
            </el-form-item>
          </el-col>
        </el-form>
      </el-row>
    </div>
    <div class="app-container flex-item">
      <div class="bg-white" style="height: 100%;">
        <el-table ref="mainTable" height="calc(100% - 52px)" :key='tableKey' :data="list" v-loading="listLoading" border
                  fit highlight-current-row stripe
                  style="width: 100%;" @row-click="rowClick" @selection-change="handleSelectionChange">
          <el-table-column type="selection" align="center" width="55"></el-table-column>
          <template v-for="(headerItem,index) in headerList">
            <el-table-column v-if="headerItem.key== 'deviceType'" :label="headerItem.description" min-width="120px"
                             :key="index">
              <template slot-scope="scope">
                <span>{{ scope.row[headerItem.key] | deviceTypeOptionsFilter }}</span>
              </template>
            </el-table-column>
            <el-table-column v-else-if="headerItem.key== 'category'" :label="headerItem.description" min-width="120px"
                             :key="index">
              <template slot-scope="scope">
                <span>{{ scope.row[headerItem.key] | deviceCategoryOptionsFilter }}</span>
              </template>
            </el-table-column>
            <el-table-column v-else-if="headerItem.key== 'manufacturingCompany'" :label="headerItem.description"
                             min-width="120px"
                             :key="index">
              <template slot-scope="scope">
                <span>{{ scope.row[headerItem.key] | manufactureCompanyOptionsFilter }}</span>
              </template>
            </el-table-column>
            <el-table-column v-else-if="headerItem.key== 'purchaseCompany'" :label="headerItem.description"
                             min-width="120px"
                             :key="index">
              <template slot-scope="scope">
                <span>{{ scope.row[headerItem.key] | purchaseCompanyOptionsFilter }}</span>
              </template>
            </el-table-column>
            <el-table-column v-else-if="headerItem.key== 'managementLevel'" :label="headerItem.description"
                             min-width="120px"
                             :key="index">
              <template slot-scope="scope">
                <span>{{ scope.row[headerItem.key] | managementOptionsFilter }}</span>
              </template>
            </el-table-column>
            <el-table-column v-else-if="headerItem.key == 'deviceStatus'" :label="headerItem.description" :key="index">
              <template slot-scope="scope">
                <el-tag :type="scope.row.deviceStatus === true ? 'success' : 'danger'" disable-transitions>
                  <span>{{ scope.row[headerItem.key] | statusFilterVal }}</span>
                </el-tag>
              </template>
            </el-table-column>
            <el-table-column v-else :label="headerItem.description" min-width="120px" :key="index">
              <template slot-scope="scope">
                <span>{{ scope.row[headerItem.key] }}</span>
              </template>
            </el-table-column>
          </template>
          <el-table-column align="center" :label="'操作'" width="120" class-name="small-padding fixed-width">
            <template slot-scope="scope">
              <el-button type="primary" size="mini" plain @click="deviceDetail(scope.row)">详情</el-button>

            </template>
          </el-table-column>
        </el-table>
        <pagination v-show="total>0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit"
                    @pagination="handleCurrentChange"/>
      </div>
    </div>
  </div>
</template>

<script>

import klSelect from '@/components/KlSelect'
import klCascader from '@/components/KlCascader'
import * as deviceMains from '@/api/deviceMain'
import * as commonApi from "@/api/common";
import waves from '@/directive/waves' // 水波纹指令
import Pagination from '@/components/Pagination'
import elDragDialog from '@/directive/el-dragDialog'

let that

export default {
  name: 'deviceMain',
  components: {
    Pagination, klSelect, klCascader
  },
  directives: {
    waves,
    elDragDialog
  },
  data() {
    return {
      multipleSelection: [], // 列表checkbox选中的值
      tableKey: 0,
      list: null,
      total: 0,
      listLoading: true,
      listQuery: { // 查询条件
        page: 1,
        limit: 20,
        deviceName: '',
        deviceModel: '',
        deviceType: '',
        deviceStatus: '',
        locationCode: '',
        location: '',
        department: '',
        key: undefined,
        appId: undefined
      },
      headerList: [{
        "key": "deviceName",
        "description": "设备名称",
        "browsable": true,
        "type": "String",
        "sort": 2
      }, {
        "key": "deviceType",
        "description": "设备类型",
        "browsable": true,
        "type": "String",
        "sort": 3
      }, {
        "key": "category",
        "description": "设备类别",
        "browsable": true,
        "type": "String",
        "sort": 4
      }, {
        "key": "deviceModel",
        "description": "设备型号",
        "browsable": true,
        "type": "String",
        "sort": 5
      }, {
        "key": "specification",
        "description": "设备规格",
        "browsable": true,
        "type": "String",
        "sort": 6
      }, {
        "key": "NetWorth",
        "description": "净值",
        "browsable": true,
        "type": "String",
        "sort": 7
      }, {
        "key": "locationCode",
        "description": "位号",
        "browsable": true,
        "type": "String",
        "sort": 7
      }, {
        "key": "location",
        "description": "位置信息",
        "browsable": true,
        "type": "String",
        "sort": 7
      }, {
        "key": "managementLevel",
        "description": "设备管理级别",
        "browsable": true,
        "type": "String",
        "sort": 8
      }, {
        "key": "manufacturingCompany",
        "description": "制造单位",
        "browsable": true,
        "type": "String",
        "sort": 8
      }, {
        "key": "purchaseCompany",
        "description": "采购单位",
        "browsable": true,
        "type": "String",
        "sort": 8
      }, {
        "key": "operationDate",
        "description": "投运日期",
        "browsable": true,
        "type": "String",
        "sort": 8
      }, {
        "key": "deviceStatus",
        "description": "设备状态",
        "browsable": true,
        "type": "Boolean",
        "sort": 99
      }],
      DeviceCategoryOptions: [],//设备类别
      DeviceTypeOptions: [],//设备类型
      ManagementOptions: [],//设备管理级别
      ManufactureCompanyOptions: [],//制造单位
      PurchaseCompanyOptions: [],//采购单位
      DeviceStatusOptions: [{
        "label": "在用",
        "value": true
      }, {
        "label": "停用",
        "value": false
      }],
      DepartmentProps: {
        // "multiple": false,
        "label": "name",
        "value": "id",
        // "children": "children",
        "expandTrigger": 'hover',
        "checkStrictly": true
      },
    }
  },
  filters: {
    statusFilter(disable) {
      const statusMap = {
        false: 'color-success',
        true: 'color-danger'
      }
      return statusMap[disable]
    },
    //状态filter
    statusFilterVal(val) {
      if (val) {
        return '在用'
      }
      return '已停用'
    },
    //设备类型filter
    deviceTypeOptionsFilter(val) {
      let found = that.DeviceTypeOptions.find(s => s.id === val);
      if (found) {
        return found.name
      }
      return val
    },
    //设备类别filter
    deviceCategoryOptionsFilter(val) {
      let found = that.DeviceCategoryOptions.find(s => s.id === val);
      if (found) {
        return found.name
      }
      return val
    },
    managementOptionsFilter(val) {
      let found = that.ManagementOptions.find(s => s.id === val);
      if (found) {
        return found.name
      }
      return val
    },
    manufactureCompanyOptionsFilter(val) {
      let found = that.ManufactureCompanyOptions.find(s => s.id === val);
      if (found) {
        return found.name
      }
      return val
    },
    purchaseCompanyOptionsFilter(val) {
      let found = that.PurchaseCompanyOptions.find(s => s.id === val);
      if (found) {
        return found.name
      }
      return val
    }
  },
  beforeCreate() {
    that = this
  },
  created() {
    this.getList()
    this.getDeviceTypeOptions()//设备类型
    this.getDeviceCategoryOptions()//设备类别
    this.getManagementOptions()//设备管理级别
    this.getManufactureCompanyOptions()//制造单位
    this.getPurchaseCompanyOptions()//采购单位
  },
  beforeMount() {
  },
  mounted() {
  },
  methods: {
    rowClick(row) {
      this.$refs.mainTable.clearSelection()
      this.$refs.mainTable.toggleRowSelection(row)
    },
    handleSelectionChange(val) {
      this.multipleSelection = val
    },
    onBtnClicked: function (domId) {
      // console.log('you click:' + domId)
      switch (domId) {
        case 'btnAdd':
          this.continueRefine()
          break
        case 'btnEdit':
          if (this.multipleSelection.length !== 1) {
            this.$message({
              message: '请选中一个进行编辑',
              type: 'error'
            })
            return
          }
          this.continueRefine(this.multipleSelection[0])
          break
        case 'btnDel':
          if (this.multipleSelection.length < 1) {
            this.$message({
              message: '至少删除一个',
              type: 'error'
            })
            return
          }
          this.handleDelete(this.multipleSelection)
          break
        default:
          break
      }
    },
    getList() {
      this.listLoading = true
      deviceMains.getList(this.listQuery).then(response => {
        this.list = response.data
        // noinspection JSUnresolvedVariable
        // response.columnHeaders.forEach((item) => {
        //   item.key = item.key.substring(0, 1).toLowerCase() + item.key.substring(1)
        // })
        // noinspection JSUnresolvedVariable
        // this.headerList = response.columnHeaders.filter(u => u.browsable)
        // console.log(`this.headerList  ${JSON.stringify(this.headerList )}`)
        // noinspection JSUnresolvedVariable
        this.total = response.count
        this.listLoading = false
      })
    },
    resetListQuery() {
      this.listQuery = { // 查询条件
        page: 1,
        limit: 20,
        deviceName: '',
        deviceModel: '',
        deviceType: '',
        deviceStatus: '',
        locationCode: '',
        location: '',
        department: '',
        key: undefined,
        appId: undefined
      }
    },
    handleFilter(reset) {
      if (reset) {
        this.resetListQuery()
      } else {
        this.listQuery.page = 1
      }
      this.getList()
    },
    handleSizeChange(val) {
      this.listQuery.limit = val
      this.getList()
    },
    handleCurrentChange(val) {
      this.listQuery.page = val.page
      this.listQuery.limit = val.limit
      this.getList()
    },
    handleModifyStatus(row) { // 模拟修改状态
      // row.disable = disable;
      this.deviceMain = Object.assign({}, row); // copy obj
      this.deviceMain.deviceStatus = !row.deviceStatus;
      // 更新提交
      const tempData = Object.assign({}, this.deviceMain);
      deviceMains.update(tempData).then((res) => {
        // noinspection JSUnresolvedVariable
        if (res.code === 200) {
          this.$notify({
            title: "成功",
            message: "操作成功",
            type: "success",
            duration: 2000,
          });
          for (const v of this.list) {
            if (v.id === this.deviceMain.id) {
              const index = this.list.indexOf(v);
              this.list.splice(index, 1, this.deviceMain);
              break;
            }
          }
        } else {
          this.$notify({
            title: "失败",
            message: "操作失败",
            type: "error",
            duration: 2000,
          });
        }
      });
    },
    //设备类型Option
    getDeviceTypeOptions() {
      if (this.DeviceTypeOptions.length <= 0) {
        commonApi.loadData('Categorys', 'LoadSelectDataSourceByType', {typeId: 'EQUIPMENT_TYPE'}).then((res) => {
          if (res.code === 200) {
            let data = res.result.map(function (item) {
              return {
                id: item.value,
                name: item.label
              };
            });
            this.DeviceTypeOptions = data
          }
        });
      }
    },
    //设备类别Option
    getDeviceCategoryOptions() {
      if (this.DeviceCategoryOptions.length <= 0) {
        commonApi.loadData('EquipmentCategorys', 'GetTreeData', {}).then((res) => {
          // noinspection JSUnresolvedVariable
          if (res.code === 200) {
            this.DeviceCategoryOptions = res.result
          }
        });
      }
    },
    //管理级别
    getManagementOptions() {
      if (this.ManagementOptions.length <= 0) {
        commonApi.loadData('Categorys', 'LoadSelectDataSourceByType', {typeId: 'Equipment_Management_Level'}).then((res) => {
          if (res.code === 200) {
            // noinspection JSUnresolvedVariable
            let data = res.result.map(function (item) {
              return {
                id: item.value,
                name: item.label
              };
            });
            // console.log(`getManagementOptions ${JSON.stringify(data)}`)
            this.ManagementOptions = data
          }
        });
      }
    },
    //制造单位
    getManufactureCompanyOptions() {
      if (this.ManufactureCompanyOptions.length <= 0) {
        commonApi.loadData('ManufactureCompanys', 'LoadSelectDataSource').then((res) => {
          if (res.code === 200) {
            // noinspection JSUnresolvedVariable
            let data = res.result.map(function (item) {
              return {
                id: item.value,
                name: item.label
              };
            });
            // console.log(`getManagementOptions ${JSON.stringify(data)}`)
            this.ManufactureCompanyOptions = data
          }
        });
      }
    },
    getPurchaseCompanyOptions() {
      if (this.PurchaseCompanyOptions.length <= 0) {
        commonApi.loadData('PurchaseCompanys', 'LoadSelectDataSource').then((res) => {
          if (res.code === 200) {
            console.log(`PurchaseCompanyOptions res is ${res.result}`)
            // noinspection JSUnresolvedVariable
            let data = res.result.map(function (item) {
              return {
                id: item.value,
                name: item.label
              };
            });
            // console.log(`getManagementOptions ${JSON.stringify(data)}`)
            this.PurchaseCompanyOptions = data
          }
        });
      }
    },
    deviceTypeChange(data) {
      if (data !== '') {
        console.log(data)
        this.listQuery.deviceType = data
      }
    },
    //联动单位
    departmentChange: function (data) {
      this.listQuery.department = data[data.length - 1]
    },
    deviceDetail(row) {
      this.$router.push('/DeviceMain/detail/' + row.id)
    }
  }
}
</script>
<style scoped>

</style>
