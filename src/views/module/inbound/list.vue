<template>
  <div class="qingwu">
    <div class="admin_main_block">
      <div class="admin_main_block_top">
        <div class="admin_main_block_left">
          <div>
            <el-button v-if="isAuth('module:inbound:form')" type="success" icon="el-icon-download" @click="$router.push({name: 'module_inbound_form'})">
              {{ $t('inventory.inbound.from') }}
            </el-button>
          </div>
        </div>
      </div>

      <div class="admin_main_block_top">
        <div class="admin_main_block_left">
          <div>
            <el-select v-model="dataForm.type" :placeholder="$t('common.please_select') + $t('inventory.type')" clearable>
              <el-option :label="$t('common.all')" value=""></el-option>
              <el-option v-for="(v,k) in typeList" :label="v.title" :key="k" :value="v.id"></el-option>
            </el-select>
          </div>
          <div>
            <el-select v-model="dataForm.category" :placeholder="$t('common.please_select') + $t('inventory.inbound.category')" clearable>
              <el-option :label="$t('common.all')" value=""></el-option>
              <el-option v-for="(v,k) in categoryList" :label="v.title" :key="k" :value="v.id"></el-option>
            </el-select>
          </div>
          <div>
            <el-date-picker format="yyyy-MM-dd HH:mm" v-model="dataForm.create_time" type="daterange" :range-separator="$t('common.to')" :start-placeholder="$t('common.start_time')" :end-placeholder="$t('common.end_time')" clearable>
            </el-date-picker>
          </div>
          <div>
            <el-button icon="el-icon-search" @click="getDataList(true)">
              {{ $t('common.search') }}
            </el-button>
          </div>
        </div>
      </div>

      <div class="admin_table_main color">
        <el-table :data="dataList" v-loading="dataListLoading" @selection-change="selectionChangeHandle">
          <el-table-column type="selection" header-align="center" align="center">
          </el-table-column>

          <el-table-column prop="id" label="#"  width="70px">
          </el-table-column>

          <el-table-column :label="$t('inventory.type')">
            <template slot-scope="scope">
              {{ scope.row.type.text }}
            </template>
          </el-table-column>

          <el-table-column :label="$t('inventory.inbound.category')" width="80">
            <template slot-scope="scope">
              {{ scope.row.category.text }}
            </template>
          </el-table-column>

          <el-table-column :label="$t('agent.nickname')" width="80">
            <template slot-scope="scope">
              <span v-if="scope.row.member">
                {{ scope.row.member.nickname }}
              </span>
              <span v-else-if="0 == scope.row.member_id">
                {{ $t('inventory.inbound.nickname') }}
              </span>
            </template>
          </el-table-column>

          <el-table-column :label="$t('agent.username')" width="80">
            <template slot-scope="scope">
              <span v-if="scope.row.member">
                {{ scope.row.member.username }}
              </span>
              <span v-else>
                {{ $t('common.empty') }}
              </span>
            </template>
          </el-table-column>

          <el-table-column prop="total" :label="$t('inventory.inbound.total')" width="80">
          </el-table-column>

          <el-table-column :label="$t('inventory.inbound.actual_total')" width="80">
            <template slot-scope="scope">
              <b class="green1">
                {{ scope.row.actual_total }}
              </b>
            </template>
          </el-table-column>

          <el-table-column :label="$t('inventory.outbound.abnormal_status')" width="80">
            <template slot-scope="scope">
              <b :class="2 == scope.row.abnormal_status.value ? 'red' : 'blue'">
                {{ scope.row.abnormal_status.text }}
              </b>
            </template>
          </el-table-column>

          <el-table-column prop="operator" :label="$t('inventory.outbound.operator')" width="80">
          </el-table-column>

          <el-table-column :label="$t('inventory.inbound.inbound_status')" width="80">
            <template slot-scope="scope">
              {{ scope.row.inbound_status.text }}
            </template>
          </el-table-column>

          <el-table-column prop="create_time" :label="$t('inventory.inbound.create_time')" width="140">
          </el-table-column>

          <el-table-column :label="$t('common.handle')" fixed="right" width="460">
            <template slot-scope="scope">
              <span>
                <el-button class="mr10i" v-if="isAuth('module:inbound:form') && scope.row.inbound_status.value != 3" type="success" icon="el-icon-download" @click="$router.push({name: 'module_inbound_form', query: {id: scope.row.id}})">
                  {{ $t('inventory.inbound.update') }}
                </el-button>
              </span>
              <span>
                <el-button v-if="isAuth('module:inbound:detail:list')" type="info" icon="el-icon-printer" @click="$router.push({name: 'module_inbound_detail_list', query: {inbound_id: scope.row.id}})">
                  {{ $t('inventory.inbound.detail_info') }}
                </el-button>

                <el-button v-if="isAuth('module:inbound:log:list') && scope.row.abnormal_status.value == 2" type="warning" icon="el-icon-tickets" @click="$router.push({name: 'module_inbound_log_list', query: {inbound_id: scope.row.id}})">
                  {{ $t('inventory.inbound.log_info') }}
                </el-button>

                <el-button v-if="isAuth('module:inbound:abnormal:list')" type="warning" icon="el-icon-question" @click="$router.push({name: 'module_inbound_abnormal_list', query: {inbound_id: scope.row.id}})">
                  {{ $t('inventory.inbound.abnormal_info') }}
                </el-button>

                <el-button v-if="isAuth('module:inbound:resource:view')" icon="el-icon-paperclip" @click="$router.push({name: 'module_inbound_resource_view', query: {inbound_id: scope.row.id}})">
                  {{ $t('inventory.log.attachment_info') }}
                </el-button>
              </span>
            </template>
          </el-table-column>
        </el-table>

        <div class="admin_table_main_pagination">
          <el-pagination
            @size-change="sizeChangeHandle"
            @current-change="currentChangeHandle"
            :page-size="pageSize"
            :total="totalPage"
            :current-page="pageIndex"
            background layout="prev, pager, next,jumper,total">
          </el-pagination>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import common from '@/views/common/base'
  export default {
    extends: common,
    data() {
      return {
        model: 'inbound',
        category_id: 0,
        typeList: [
          {'id': 1, 'title': '打印机'},
          {'id': 2, 'title': '墨盒'},
          {'id': 3, 'title': '纸张'},
        ],
        categoryList: [
          {'id': 1, 'title': '新入库'},
          {'id': 2, 'title': '损坏入库'},
          {'id': 3, 'title': '返修入库'},
        ],
        dataForm: [
          'type',
          'category',
          'create_time',
        ]
      };
    },
    created() {
      this.getDataList()
    }
  };
</script>
