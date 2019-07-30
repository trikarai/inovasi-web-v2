<template>
  <div>
    <v-container>
      <notification-alert v-bind:err_msg="err_msg" v-bind:status="status" />

      <v-dialog v-model="dialogDetail" max-width="350">
        <v-card>
          <v-card-title class="headline">{{singleData.name}}</v-card-title>
          <v-card-text>
            {{singleData.name}}
            <br />
            {{singleData.sessionDuration}}
            <br />
            {{singleData.FormForParticipantFeedback.name}}
            <br />
            {{singleData.FormForMentorFeedback.name}}
            <br />
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn small fab color="red darken-1" text @click="dialogDetail = false">
              <v-icon>close</v-icon>
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <v-btn @click="openAdd()" color="blue" style="left: -8px">
        <v-icon>add</v-icon>
        {{ $vuetify.t('$vuetify.action.add') }} Mentoring Event
      </v-btn>

      <v-data-table
        dark
        :loading="loader"
        :headers="headers"
        :items="mentoring.list"
        class="elevation-1"
      >
        <template v-slot:items="props">
          <td>{{ props.item.name }}</td>
          <td class="text-xs-right">
            <v-btn @click="openDetail(props.item.id)" small fab>
              <v-icon>search</v-icon>
            </v-btn>

            <v-btn color="red" @click="deleteAct(props.item.id)" small>
              <v-icon small left>delete</v-icon>
              {{ $vuetify.t('$vuetify.action.delete') }}
            </v-btn>

            <v-expand-transition>
              <div v-show="props.item.id == selectedIndex">
                {{ $vuetify.t('$vuetify.action.confirmationtodelete') }}
                <v-btn @click="deleteData(props.item.id)" color="red">
                  <v-icon></v-icon>
                  {{ $vuetify.t('$vuetify.action.yes') }}
                </v-btn>
                <v-btn @click="deleteAct(null)">
                  <v-icon></v-icon>
                  {{ $vuetify.t('$vuetify.action.cancel') }}
                </v-btn>
              </div>
            </v-expand-transition>
          </td>
        </template>
      </v-data-table>
    </v-container>

    <mentoring-form
      v-if="dialogForm"
      :edit="edit"
      :view="view"
      @close="dialogForm = false"
      @refresh="refresh"
    />
  </div>
</template>
<script>
import net from "@/config/httpclient";
import notif from "@/config/alerthandling";
import Notification from "@/components/Notification";
import MentoringForm from "./mentoring/MentoringForm";

export default {
  components: {
    MentoringForm,
    "notification-alert": Notification
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
      dialogDetail: false,
      dialogForm: false,
      edit: false,
      view: false,
      expand: false,
      dataId: "",
      selectedIndex: null,
      mentoring: { total: 0, list: [] },
      singleData: {
        id: "",
        name: "",
        sessionDuration: "",
        FormForParticipantFeedback: { name: "" },
        FormForMentorFeedback: { name: "" }
      },
      headers: [
        {
          text: "Name",
          align: "left",
          sortable: false,
          value: "order"
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
        .getData(
          this,
          "/administrator/programmes/" +
            this.$route.params.programId +
            "/mentorings"
        )
        .then(res => {
          if (res.data.data) {
            this.mentoring = res.data.data;
          } else {
            this.mentoring.list = [];
          }
        })
        .catch(error => {
          console.log(error);
          notif.showError(this, error);
        })
        .finally(() => {
          this.loader = false;
        });
    },
    getSingleData: function(id) {
      this.loader = true;
      net
        .getData(
          this,
          "/administrator/programmes/" +
            this.$route.params.programId +
            "/mentorings/" +
            id
        )
        .then(res => {
          if (res.data.data) {
            this.singleData = res.data.data;
          } else {
            this.singleData = null;
          }
        })
        .catch(error => {
          console.log(error);
          this.singleData = error.status;
          notif.showError(this, error);
        })
        .finally(() => {
          this.loader = false;
        });
    },
    openAdd: function() {
      this.dialogForm = true;
      this.edit = false;
    },
    openDetail: function(id) {
      this.dialogDetail = true;
      this.getSingleData(id);
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
          "/administrator/programmes/" +
            this.$route.params.programId +
            "/mentorings/" +
            id
        )
        .then(res => {
          console.log(res);
          this.refresh();
        })
        .catch(error => {
          console.log(error);
          notif.showError(this, error);
        })
        .finally(function() {
          this.selectedIndex = null;
        });
    },
    refresh: function() {
      this.dialogForm = false;
      this.getDataList();
    }
  }
};
</script>