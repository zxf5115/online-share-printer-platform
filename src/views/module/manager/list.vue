<template>
  <div class="qingwu">
    <div class="admin_main_block">
      <div class="admin_main_block_top">
        <div class="admin_main_block_right">
          <div>
            <el-button v-if="isAuth('module:manager:delete')" type="danger" icon="el-icon-delete" @click="deleteHandle()">
              {{ $t('common.batch_delete') }}
            </el-button>
          </div>
        </div>
      </div>

      <div class="admin_main_block_top">
        <div class="admin_main_block_left">
          <div>
            <el-input v-model="dataForm.username" :placeholder="$t('common.please_input') + $t('manager.username')" clearable>
            </el-input>
          </div>
          <div>
            <el-input v-model="dataForm.nickname" :placeholder="$t('common.please_input') + $t('manager.nickname')" clearable>
            </el-input>
          </div>
          <div>
            <el-button icon="el-icon-search" @click="getDataList(true)">
              {{ $t('common.search') }}
            </el-button>
          </div>
        </div>
      </div>
      <div class="admin_table_main">
        <el-table :data="dataList" v-loading="dataListLoading" @selection-change="selectionChangeHandle">
          <el-table-column type="selection" header-align="center" align="center">
          </el-table-column>

          <el-table-column prop="id" :label="$t('common.id')" width="70">
          </el-table-column>

          <el-table-column :label="$t('manager.agent_name')" width="120">
            <template slot-scope="scope">
              <span v-if="scope.row.parent">
                {{ scope.row.parent.nickname }}
              </span>
              <span v-else>
                {{ $t('common.empty') }}
              </span>
            </template>
          </el-table-column>

          <el-table-column :label="$t('manager.info')" min-width="260">
            <template slot-scope="scope">
              <dl class="table_dl">
                <dt>
                  <el-avatar :size="45" :src="scope.row.avatar">
                    <img src="@/assets/images/default/circle.png"/>
                  </el-avatar>
                </dt>
                <dd class="table_dl_dd_all_30">
                  {{ $t('manager.nickname') }}： {{ scope.row.nickname }}
                </dd>
                <dd class="table_dl_dd_all_16_gray">
                  {{ $t('manager.create_time') }}： {{ scope.row.create_time }}
                </dd>
              </dl>
            </template>
          </el-table-column>

          <el-table-column prop="username" :label="$t('manager.username')" width="100">
            <template slot-scope="scope">
              {{ scope.row.username | emptyData }}
            </template>
          </el-table-column>

          <el-table-column :label="$t('manager.asset.money')" width="90">
            <template slot-scope="scope" v-if="scope.row.asset">
              {{ scope.row.asset.money }}
            </template>
          </el-table-column>

          <el-table-column :label="$t('manager.asset.proportion')" width="80">
            <template slot-scope="scope" v-if="scope.row.parent && scope.row.parent.asset">
              {{ scope.row.parent.asset.manger_proportion }}
            </template>
          </el-table-column>

          <el-table-column prop="printer_total" :label="$t('manager.printer_total')" width="80">
          </el-table-column>

          <el-table-column :label="$t('manager.asset.order_total')" width="80">
            <template slot-scope="scope" v-if="scope.row.asset">
              {{ scope.row.asset.order_total }}
            </template>
          </el-table-column>

          <el-table-column :label="$t('common.area')" width="180">
            <template slot-scope="scope" v-if="scope.row.archive">
              {{ scope.row.archive.province_id.text }}
            </template>
          </el-table-column>

          <el-table-column prop="status" :label="$t('manager.status')" width="100">
            <template slot-scope="scope">
              <el-switch
                v-model="scope.row.status.value"
                :active-value="1"
                :inactive-value="2"
                @change="handleStatus($event, scope.row.id, 'status')">
              </el-switch>
            </template>
          </el-table-column>

          <el-table-column :label="$t('common.handle')" fixed="right" width="300">
            <template slot-scope="scope">
              <el-button v-if="isAuth('module:manager:view')" type="info" icon="el-icon-view" @click="$router.push({name: 'module_manager_view', query: {id: scope.row.id}})">
                {{ $t('common.view') }}
              </el-button>

              <el-button v-if="isAuth('module:manager:bank:form')" type="primary" icon="el-icon-edit" @click="$router.push({name: 'module_manager_bank_form', query: {id: scope.row.id}})">
                {{ $t('agent.bank_card') }}
              </el-button>

              <el-button v-if="isAuth('module:manager:delete')" type="danger" icon="el-icon-delete" @click="deleteHandle(scope.row.id)">
                {{ $t('common.delete') }}
              </el-button>
            </template>
          </el-table-column>
        </el-table>
        <div class="admin_table_main_pagination">
          <el-pagination
            @size-change="sizeChangeHandle"
            @current-change="currentChangeHandle"
            :page-sizes="[10, 50, 100, 500]"
            :page-size="pageSize"
            :total="totalPage"
            :current-page="pageIndex"
            background layout="prev, pager, next, jumper, total, sizes">
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
        model: 'manager',
        provinceList: [],
        dataForm: [
          'username',
          'nickname',
          'parent_nickname',
          'archive_province_id',
        ],
      };
    },
    created() {
      this.dataForm.parent_id = this.$route.query.parent_id;

      this.getDataList();
    }
  };
</script>

