<template>
  <div>
    <v-container>
      <notification-alert v-bind:err_msg="err_msg" v-bind:status="status" />
      <!-- {{res}}<br> -->
      <v-btn @click="openAdd()" color="blue" class="mb-3">
        <v-icon>add</v-icon>
        {{ $vuetify.lang.t('$vuetify.action.add') }} Phase
      </v-btn>
      <v-data-table
        dark
        :headers="headers"
        :loading="loader"
        :items="phase.list"
        class="elevation-1"
      >
        <template v-slot:item.action="{item}">
          <v-btn @click="openEdit(item.id)" small class="ma-1">
            <v-icon small>edit</v-icon>
            {{ $vuetify.lang.t('$vuetify.action.edit') }}
          </v-btn>
          <v-btn small dark color="warning" @click="deleteAct(item.id)">
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
    <PhaseForm
      :data="singlePhase"
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
import notif from "@/config/alerthandling";
import PhaseForm from "./AddPhase";

export default {
  components: {
    "notification-alert": Notification,
    PhaseForm
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
      err_msg: { details: [""] },
      loader: false,
      dialogDel: false,
      dialogForm: false,
      edit: false,
      view: false,
      expand: false,
      dataId: "",
      selectedIndex: null,
      phase: { total: 0, list: [] },
      singlePhase: { id: "", name: "" },
      headers: [
        {
          text: "Order",
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
        { text: "Actions", value: "action", sortable: false, align:"right" }
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
        .getData(
          this,
          "/administrator/curriculums/" +
            this.$route.params.curriculumId +
            "/phase-plans"
        )
        .then(res => {
          if (res.data.data) {
            this.phase = res.data.data;
          } else {
            this.phase.list = [];
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
      this.singlePhase.id = index;
    },
    openAdd: function() {
      this.dialogForm = true;
      this.view = false;
      this.edit = false;
      this.singlePhase = { id: "", name: "" };
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
        .deleteData(
          this,
          "/administrator/curriculums/" +
            this.$route.params.curriculumId +
            "/phase-plans/" +
            id
        )
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
