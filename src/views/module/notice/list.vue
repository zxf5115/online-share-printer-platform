<template>
  <div class="qingwu">
    <div class="admin_main_block">
      <div class="admin_main_block_top">
        <div class="admin_main_block_left">
          <div>
            <router-link :to="{name: 'module_notice_form'}">
              <el-button v-if="isAuth('module:notice:form')" type="success" icon="el-icon-plus">
                {{ $t('common.create') }}
              </el-button>
            </router-link>
          </div>
          <div>
            <el-button v-if="isAuth('module:notice:category:list')" icon="el-icon-price-tag" @click="$router.push({name: 'module_notice_category_list'})">
              {{ $t('notice.category.list') }}
            </el-button>
          </div>
        </div>

        <div class="admin_main_block_right">
          <div>
            <el-button v-if="isAuth('module:notice:delete')" type="danger" icon="el-icon-delete" @click="deleteHandle()">
              {{ $t('common.batch_delete') }}
            </el-button>
          </div>
        </div>
      </div>
      <div class="admin_main_block_top">
        <div class="admin_main_block_left">
          <div>
            <el-input v-model="dataForm.content" :placeholder="$t('common.please_input') + $t('notice.content')" clearable>
            </el-input>
          </div>
          <div>
            <el-select v-model="dataForm.category_id" :placeholder="$t('common.please_select') + $t('notice.category.title')" clearable>
              <el-option :label="$t('common.all')" value=""></el-option>
              <el-option v-for="(v,k) in categoryList" :label="v.title" :key="k" :value="v.id"></el-option>
            </el-select>
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

          <el-table-column prop="id" label="#" width="70px">
          </el-table-column>

          <el-table-column prop="category_id" :label="$t('notice.category.title')" width="120">
            <template slot-scope="scope" v-if="scope.row.category">
              {{ scope.row.category.title }}
            </template>
          </el-table-column>

          <el-table-column prop="content" :label="$t('notice.content')">
          </el-table-column>

          <el-table-column :label="$t('notice.delivery_status')" width="80">
            <template slot-scope="scope">
              {{ scope.row.delivery_status.text }}
            </template>
          </el-table-column>

          <el-table-column prop="create_time" :label="$t('common.create_time')" width="140">
          </el-table-column>

          <el-table-column :label="$t('common.handle')" fixed="right" width="280">
            <template slot-scope="scope">
              <el-button v-if="isAuth('module:notice:form')" type="primary" icon="el-icon-edit" @click="$router.push({name: 'module_notice_form', query: {id: scope.row.id}})">
                {{ $t('common.update') }}
              </el-button>

              <el-button v-if="isAuth('module:notice:status') && scope.row.delivery_status.value == 2" icon="el-icon-s-promotion" @click="handleStatus(scope.row.id)">
                {{ $t('notice.send') }}
              </el-button>

              <el-button v-if="isAuth('module:notice:delete')" type="danger" icon="el-icon-delete" @click="deleteHandle(scope.row.id)">
                {{ $t('common.delete') }}
              </el-button>
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
        model: 'notice',
        categoryList: [],
        dataForm: [
          'category_id',
          'title'
        ]
      };
    },
    methods: {
      loadCategoryList () {
        this.$http({
          url: this.$http.adornUrl('/notice/category/select'),
          method: 'get'
        }).then(({data}) => {
          if (data && data.status === 200) {
            this.categoryList = data.data
          } else {
            this.$notice.error(this.$t(data.notice))
          }
        })
      },
      // 改变状态
      handleStatus(id) {
        this.$http({
          url: this.$http.adornUrl('/notice/status'),
          method: 'post',
          data: {
            id: id
          }
        }).then(({data}) => {
          if (data && data.status === 200) {
            this.$message({
              message: this.$t('common.handle_success'),
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.getDataList()
              }
            })
          } else {
            this.$message.error(this.$t(data.message))
          }
        })
      },
    },
    created() {
      this.getDataList();

      this.loadCategoryList();
    }
  };
</script>
