<template>
  <div>
    <v-container>
      <notification-alert v-bind:err_msg="err_msg" v-bind:status="status"/>
      <!-- {{res}}<br> -->
      <v-btn @click="openAdd()" color="primary" class="mb-3">
        <v-icon>add</v-icon>
        {{ $vuetify.lang.t('$vuetify.action.add') }} Section
      </v-btn>
      <v-dialog v-model="loader" hide-overlay persistent width="300">
        <v-card color="primary" dark>
          <v-card-text>
            {{ $vuetify.lang.t('$vuetify.info.standby') }}
            <v-progress-linear indeterminate color="white" class="mb-0"></v-progress-linear>
          </v-card-text>
        </v-card>
      </v-dialog>
      <v-data-table dark :headers="headers" :items="field.list" class="elevation-1">
        <template v-slot:items="props">
          <td>{{ props.item.id }}</td>
          <td>{{ props.item.name }}</td>
          <td class="text-xs-right">
            <v-btn @click="openEdit(props.index)" small>
              <v-icon small>edit</v-icon>
              {{ $vuetify.lang.t('$vuetify.action.edit') }}
            </v-btn>
            <v-btn small dark color="warning" @click="deleteAct(props.index)">
              <v-icon small>delete</v-icon>
              {{ $vuetify.lang.t('$vuetify.action.delete') }}
            </v-btn>
            <v-expand-transition>
              <div v-show="props.index == selectedIndex">
                {{ $vuetify.lang.t('$vuetify.action.confirmationtodelete') }}
                <v-btn @click="deleteData(props.item.id)" color="red">
                  <v-icon></v-icon>
                  {{ $vuetify.lang.t('$vuetify.action.yes') }}
                </v-btn>
                <v-btn @click="deleteAct(null)">
                  <v-icon></v-icon>
                  {{ $vuetify.lang.t('$vuetify.action.cancel') }}
                </v-btn>
              </div>
            </v-expand-transition>
          </td>
        </template>
      </v-data-table>
    </v-container>
    <SectionForm
      :data="singleData"
      :edit="edit"
      :view="view"
      v-if="dialogForm"
      @close="dialogForm = false"
      @refresh="refresh()"
    />
  </div>
</template>
<script>
import net from "@/config/httpclient";
import Notification from "@/components/Notification";
import SectionForm from "./AddSection";

export default {
  components: {
    "notification-alert": Notification,
     SectionForm
  },
  data() {
    return {
      res: "",
      status: {
        success: false,
        error: false,
        info: false,
        warning: false
      },
      err_msg: {details:[""]},
      loader: false,
      dialogDel: false,
      dialogForm: false,
      edit: false,
      view: false,
      expand: false,
      dataId: "",
      selectedIndex: null,
      field: { total: 0, list: [] },
      singleData: { id: "", name: "" },
      headers: [
        {
          text: "Id",
          align: "left",
          sortable: true,
          value: "order"
        },
        {
          text: "Name",
          align: "left",
          sortable: false,
          value: "name"
        },
        { text: "", value: "id", sortable: false }
      ]
    };
  },
  mounted: function() {
    this.getDataList();
  },
  methods: {
    getDataList: function() {
      this.loader = true;
      net
        .getData(this, "/administrator/forms/" + this.$route.params.formId + "/sections")
        .then(
          res => {
            if(res.data.data){
              this.field = res.data.data;
            }else{
              this.field.list = []; 
            }
          },
          error => {
            
            this.err_msg = error.body.meta;
            this.status.error = true;
          }
        )
        .catch(function() {
        })
        .finally(()=> {
          this.loader = false;
        });
    },
    openEdit: function(index) {
      this.dialogForm = true;
      this.view = false;
      this.edit = true;
      this.singleData = this.field.list[index];
    },
    openAdd: function() {
      this.dialogForm = true;
      this.view = false;
      this.edit = false;
      this.singleData = { id: "", name: "" };
    },
    deleteAct: function(id) {
      if (this.selectedIndex == id) {
        this.selectedIndex = null;
      } else {
        this.selectedIndex = id;
      }
    },
    deleteData: function(id) {
      net
        .deleteData(this, "/administrator/forms/" + this.$route.params.formId + "/sections/" + id)
        .then()
        .catch(function() {})
        .finally(()=> {
          this.selectedIndex = null;
          this.refresh();
        });
    },
    refresh: function() {
      this.dialogForm = false;
      this.getDataList();
    }
  }
};
</script>
