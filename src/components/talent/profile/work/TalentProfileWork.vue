<template>
  <div>
    <v-container>
      <notification-alert v-bind:err_msg="err_msg" v-bind:status="status" />
      <v-btn dark @click="openAdd()" color="primary">
        <v-icon>add</v-icon>
        {{ $vuetify.lang.t('$vuetify.action.add') }} Work Experiences
      </v-btn>
      <!-- {{data.list}} -->
      <v-data-table :headers="headers" :items="data.list" :loading="loader" class="elevation-1 mt-2">
        <template v-slot:item.action="{ item }">
          <v-flex class="text-end">
            <v-btn class="ma-2" @click="openEdit(item.id)" small>
              <v-icon small left>edit</v-icon>
              {{ $vuetify.lang.t('$vuetify.action.edit') }}
            </v-btn>
            <v-btn class="ma-2" small dark color="warning" @click="deleteAct(item.id)">
              <v-icon small left>delete</v-icon>
              {{ $vuetify.lang.t('$vuetify.action.delete') }}
            </v-btn>
            <v-expand-transition>
              <div v-show="item.id == selectedIndex">
                <div>
                   <v-icon>warning</v-icon> <span> {{ $vuetify.lang.t('$vuetify.action.confirmationtodelete') }}</span>
                </div>
                <v-btn class="ma-2" dark small @click="deleteData(item.id)" color="red">
                  <v-icon></v-icon>
                  {{ $vuetify.lang.t('$vuetify.action.yes') }}
                </v-btn>
                <v-btn class="ma-2" text small @click="deleteAct(null)">
                  <v-icon></v-icon>
                  {{ $vuetify.lang.t('$vuetify.action.cancel') }}
                </v-btn>
              </div>
            </v-expand-transition>
          </v-flex>
        </template>
      </v-data-table>
    </v-container>

    <WorkForm
      :data="singleData"
      v-bind:edit="edit"
      v-bind:view="view"
      v-if="dialogForm"
      @close="dialogForm = false"
      @refresh="refresh()"
    />
  </div>
</template>
<script>
import net from "@/config/httpclient";
import notif from "@/config/alerthandling";
import WorkForm from "./WorkingForm";
import Notification from "@/components/Notification";

export default {
  components: {
    WorkForm,
    "notification-alert": Notification
  },
  data() {
    return {
      status: {
        success: false,
        error: false,
        info: false,
        warning: false
      },
      err_msg: { details: [""] },
      loader: false,
      dialogDel: false,
      dialogForm: false,
      edit: false,
      view: false,
      expand: false,
      dataId: "",
      selectedIndex: null,
      data: { total: 0, list: [] },
      singleData: { id: "", name: "" },
      headers: [
        {
          text: "Company Name",
          align: "left",
          sortable: false,
          value: "companyName"
        },
        {
          text: "Position",
          align: "left",
          sortable: false,
          value: "position"
        },
        { text: "", value: "action", sortable: false }
      ]
    };
  },
  mounted: function() {
    this.getDataList();
  },
  methods: {
    getDataList: function() {
      this.loader = true;
      notif.reset(this);
      net
        .getData(this, "/talent/working-experiences")
        .then(res => {
          if (res.data.data) {
            this.data = res.data.data;
          } else {
            this.data.list = [];
          }
        })
        .catch(error => {
          notif.showError(this, error);
        })
        .finally(() => {
          this.loader = false;
        });
    },
    openEdit: function(index) {
      this.dialogForm = true;
      this.view = false;
      this.edit = true;
      this.singleData.id = index;
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
      notif.reset(this);
      net
        .deleteData(this, "/talent/working-experiences/" + id)
        .then()
        .catch(error => {
          notif.showError(this, error);
        })
        .finally(() => {
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
