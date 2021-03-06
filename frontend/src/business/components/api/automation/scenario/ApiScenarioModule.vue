<template>
  <div>

    <ms-node-tree
      v-loading="result.loading"
      :tree-nodes="data"
      :type="'edit'"
      @add="add"
      @edit="edit"
      @drag="drag"
      @remove="remove"
      @nodeSelectEvent="nodeChange"
      ref="nodeTree">

      <template v-slot:header>
        <el-input style="width: 275px;" :placeholder="$t('test_track.module.search')" v-model="condition.filterText"
                  size="small">
          <template v-slot:append>
            <el-button icon="el-icon-folder-add" @click="addScenario"/>
          </template>
        </el-input>
        <module-trash-button :condition="condition" :exe="enableTrash"/>
      </template>

    </ms-node-tree>

    <ms-add-basis-scenario
      @saveAsEdit="saveAsEdit"
      @refresh="refresh"
      ref="basisScenario"/>
  </div>

</template>

<script>
  import SelectMenu from "../../../track/common/SelectMenu";
  import MsAddBasisScenario from "@/business/components/api/automation/scenario/AddBasisScenario";
  import {getCurrentProjectID} from "@/common/js/utils";
  import MsNodeTree from "../../../track/common/NodeTree";
  import {buildNodePath} from "../../definition/model/NodeTree";
  import ModuleTrashButton from "../../definition/components/module/ModuleTrashButton";

  export default {
    name: 'MsApiScenarioModule',
    components: {
      ModuleTrashButton,
      MsNodeTree,
      MsAddBasisScenario,
      SelectMenu,
    },
    data() {
      return {
        result: {},
        condition: {
          filterText: "",
          trashEnable: false
        },
        projectId: "",
        data: [],
        currentModule: undefined,
      }
    },
    mounted() {
      this.projectId = getCurrentProjectID();
      this.list();
    },
    watch: {
      'condition.filterText'(val) {
        this.$refs.nodeTree.filter(val);
      },
      'condition.trashEnable'() {
        this.$emit('enableTrash', this.condition.trashEnable);
      },
    },
    methods: {

      list() {
        if (this.projectId) {
          this.result = this.$get("/api/automation/module/list/" + this.projectId + "/", response => {
            if (response.data != undefined && response.data != null) {
              this.data = response.data;
              let moduleOptions = [];
              this.data.forEach(node => {
                buildNodePath(node, {path: ''}, moduleOptions);
              });
              this.$emit('setModuleOptions', moduleOptions);
            }
          });
        }
      },
      edit(param) {
        param.projectId = this.projectId;
        param.protocol = this.condition.protocol;
        this.$post("/api/automation/module/edit", param, () => {
          this.$success(this.$t('commons.save_success'));
          this.list();
          this.refresh();
        }, (error) => {
          this.list();
        });
      },
      add(param) {
        param.projectId = this.projectId;
        param.protocol = this.condition.protocol;
        this.$post("/api/automation/module/add", param, () => {
          this.$success(this.$t('commons.save_success'));
          this.list();
        }, (error) => {
          this.list();
        });
      },
      remove(nodeIds) {
        this.$post("/api/automation/module/delete", nodeIds, () => {
          this.list();
          this.refresh();
        }, (error) => {
          this.list();
        });
      },
      drag(param, list) {
        this.$post("/api/automation/module/drag", param, () => {
          // this.$post("/api/module/pos", list); //todo 排序
          this.list();
        }, (error) => {
          this.list();
        });
      },
      nodeChange(node, nodeIds, pNodes) {
        this.currentModule = node.data;
        this.condition.trashEnable = false;
        if (node.data.id === 'root') {
          this.$emit("nodeSelectEvent", node, [], pNodes);
        } else {
          this.$emit("nodeSelectEvent", node, nodeIds, pNodes);
        }
      },
      // exportAPI() {
      //   this.$emit('exportAPI');
      // },
      // debug() {
      //   this.$emit('debug');
      // },
      saveAsEdit(data) {
        this.$emit('saveAsEdit', data);
      },
      refresh() {
        this.$emit("refreshTable");
      },
      addScenario() {
        this.$refs.basisScenario.open(this.currentModule);
      },
      enableTrash() {
        this.condition.trashEnable = true;
      }
    }
  }
</script>

<style scoped>
  .node-tree {
    margin-top: 15px;
    margin-bottom: 15px;
  }

  .ms-el-input {
    height: 25px;
    line-height: 25px;
  }

  .custom-tree-node {
    flex: 1 1 auto;
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 14px;
    padding-right: 8px;
    width: 100%;
  }

  .father .child {
    display: none;
  }

  .father:hover .child {
    display: block;
  }

  .node-title {
    width: 0;
    text-overflow: ellipsis;
    white-space: nowrap;
    flex: 1 1 auto;
    padding: 0 5px;
    overflow: hidden;
  }

  .node-operate > i {
    color: #409eff;
    margin: 0 5px;
  }

  /deep/ .el-tree-node__content {
    height: 33px;
  }

  .ms-api-buttion {
    width: 30px;
  }
</style>
