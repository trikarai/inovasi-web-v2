<template>
  <transition name="slide" mode="out-in">
    <div>
      <v-container>
        <notification-alert v-bind:err_msg="err_msg" v-bind:status="status"/>
        <v-btn @click="openAdd()" color="primary" class="mb-3">
          <v-icon>add</v-icon>
          {{ $vuetify.lang.t('$vuetify.action.add') }} Administrator
        </v-btn>
        <v-data-table dark :headers="headers" :loading="loader" :items="admin.list" class="elevation-1">
          <template v-slot:item.action="{item}">
              <v-btn @click="openDetail(item.id)" small class="ma-1">
                <v-icon small>search</v-icon>
                {{ $vuetify.lang.t('$vuetify.action.view') }}
              </v-btn>
              <v-btn @click="openEdit(item.id)" small v-if="item.id == authData.data.id" class="ma-1">
                <v-icon small>edit</v-icon>
                {{ $vuetify.lang.t('$vuetify.action.edit') }} {{ $vuetify.lang.t('$vuetify.profile.profile') }}
              </v-btn>
              <v-btn @click="openPassword(item.id)" small v-if="item.id == authData.data.id">
                <v-icon small>edit</v-icon>
                {{ $vuetify.lang.t('$vuetify.action.edit') }} {{ $vuetify.lang.t('$vuetify.profile.password') }}
              </v-btn>
              <v-btn small dark color="warning" @click="deleteAct(item.id)" v-if="item.id != authData.data.id">
                <v-icon small>delete</v-icon>
                {{ $vuetify.lang.t('$vuetify.action.delete') }}
              </v-btn>
              <v-expand-transition>
                <div v-show="item.id == selectedIndex">
                  {{ $vuetify.lang.t('$vuetify.action.confirmationtodelete') }}
                  <v-btn @click="deleteData(item.id)" color="red" class="ma-2">
                    <v-icon></v-icon>
                    {{ $vuetify.lang.t('$vuetify.action.yes') }}
                  </v-btn>
                  <v-btn @click="deleteAct(null)">
                    <v-icon></v-icon>
                    {{ $vuetify.lang.t('$vuetify.action.cancel') }}
                  </v-btn>
                </div>
              </v-expand-transition>
          </template>
        </v-data-table>
      </v-container>

      <AdminForm
        :data="singleData"
        v-bind:edit="edit"
        v-bind:view="view"
        v-if="dialogForm"
        @refresh="refresh"
        @close="dialogForm = false"
      />
      <PasswordForm
        :data="singleData"
        v-bind:edit="edit"
        v-bind:view="view"
        v-if="dialogPassword"
        @refresh="refresh"
        @close="dialogPassword = false"
      />
    </div>
  </transition>
</template>
<script>
import net from "@/config/httpclient";
import auth from "@/config/auth"
import AdminForm from "./admin/AddAdmin";
import PasswordForm from "./admin/EditPassword";
import Notification from "@/components/Notification";

export default {
  components: {
    AdminForm,
    PasswordForm,
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
      authData: "",
      singleData: { id: "", name: "" },
      err_msg: {details:[""]},
      loader: false,
      dialogDel: false,
      dialogForm: false,
      dialogPassword: false,
      edit: false,
      view: false,
      expand: false,
      adminid: "",
      selectedIndex: null,
      headers: [
        {
          text: "Name",
          align: "left",
          sortable: false,
          value: "name"
        },
        { text: "Actions", value: "action", sortable: false, align: "right" }
      ],
      admin: {
        total: 0,
        list: []
      }
    };
  },
  created: function(){
    this.authData = JSON.parse(auth.getAuthData());
  },
  mounted: function() {
    this.getDataList();
  },
  methods: {
    getDataList: function() {
      this.loader = true;
      net
        .getData(this, "/administrator/administrators")
        .then(
          res => {
            if (res.data.data) {
              this.admin = res.data.data;
            } else {
              this.admin.list = [];
            }
          },
          error => {
            
            this.err_msg = error.body.meta;
            this.status.error = true;
          }
        )
        .catch(function() {})
        .finally(()=> {
          this.loader = false;
        });
    },
    openDetail: function(index) {
      this.dialogForm = true;
      this.view = true;
      this.edit = true;
      this.singleData.id = index;
    },
    openEdit: function(index) {
      this.dialogForm = true;
      this.view = false;
      this.edit = true;
      this.singleData.id = index;
    },
    openPassword: function(index) {
      this.dialogPassword = true;
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
      net
        .deleteData(this, "/administrator/administrators/" + id)
        .then()
        .catch(function() {})
        .finally(()=> {
          this.selectedIndex = null;
          this.refresh();
        });
    },
    refresh: function() {
      this.dialogForm = false;
      this.dialogPassword = false;
      this.getDataList();
    }
  }
};
</script>
<style scoped>
.slide-fade-enter-active {
  transition: all 0.3s ease;
}
.slide-fade-leave-active {
  transition: all 0.8s cubic-bezier(1, 0.5, 0.8, 1);
}
.slide-fade-enter,
.slide-fade-leave-to {
  transform: translateX(10px);
  opacity: 0;
}
</style>