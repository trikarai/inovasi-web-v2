<template>
  <div>
    <v-container>
      <notification-alert ref="notif" v-bind:err_msg="err_msg" v-bind:status="status" />
      <form-collaboration v-model="collaboratorForm" @setUpCollaboration="setUpCollaboration"></form-collaboration>

      <loader-dialog v-model="loader"></loader-dialog>

      <v-flex>
        <div style="display:flex;" class="mb-5">
          <div>
            <h4>{{canvasName}}</h4>
          </div>
          <div v-if="checkDashboard">
            <v-scale-transition>
              <template v-if="!selectedDel">
                <v-btn v-if="!isEdit" x-small fab class="ml-4" @click="openCollaborator()">
                  <v-icon>share</v-icon>
                </v-btn>
              </template>
            </v-scale-transition>
          </div>
        </div>
        <template v-if="checkDashboard">
          <v-btn
            small
            @click="isEdit = !isEdit"
            color="primary"
            v-if="isCanvas"
            v-show="!selectedDel"
          >
            <span v-show="isEdit">
              <v-icon left>clear</v-icon>Cancel
            </span>
            <span v-show="!isEdit">
              <v-icon left>edit</v-icon>edit
            </span>
          </v-btn>
          <v-fade-transition>
            <template v-if="!isEdit">
              <v-btn
                small
                dark
                @click="selectedDel = !selectedDel"
                color="warning"
                class="ml-2"
                v-if="isCanvas"
              >
                <v-icon left>delete</v-icon>delete
              </v-btn>
            </template>
          </v-fade-transition>
        </template>
        <v-expand-transition>
          <v-layout wrap v-if="isEdit">
            <v-flex>
              <v-btn small dark class="mt-3" @click.prevent="update" color="blue">
                <v-icon left>save</v-icon>save
              </v-btn>
            </v-flex>
          </v-layout>
        </v-expand-transition>
        <v-expand-transition>
          <v-layout class="mt-5 ml-3" wrap v-if="selectedDel">
            <v-flex>
              <v-icon>warning</v-icon>
              {{ $vuetify.lang.t('$vuetify.action.confirmationtodelete') }}
              <br />
              <v-btn x-small dark class="ma-2" color="red" @click="deleteCanvas">Yes</v-btn>
              <v-btn text x-small class="ma-2 ml-0" @click="selectedDel = !selectedDel">Cancel</v-btn>
            </v-flex>
          </v-layout>
        </v-expand-transition>
      </v-flex>
    </v-container>
    <v-form v-model="valid" ref="form2">
      <v-container
        style="display: grid; justify-items: stretch; grid-gap: 5px 5px;"
        v-if="isCanvas"
      >
        <template v-if="!isEdit">
          <template v-for="field in canvas.fields" id="viewCanvas">
            <div :key="field.id" :style="'grid-area:' + getGridPosition(field.field.position) + ''">
              <v-card class="elevation-3" :hover="hover" height="100%">
                <v-card-title class="title">
                  <span style="color: #00667f !important">{{field.field.name}}</span>
                </v-card-title>
                <div class="lain"></div>
                <v-card-text>{{field.value}}</v-card-text>
              </v-card>
            </div>
          </template>
        </template>
        <!-- grid edit view -->
        <template v-if="isEdit">
          <template v-for="(field, index) in canvas.fields" id="editCanvas">
            <div :key="field.id" :style="'grid-area:' + getGridPosition(field.field.position) + ''">
              <v-card class="elevation-3" :hover="hover" height="100%">
                <v-card-title class="title">
                  <span style="color: #00667f !important">{{field.field.name}}</span>
                </v-card-title>
                <div class="lain"></div>
                <v-card-text>
                  <fieldcanedit-modul v-bind:index="index" v-bind:fields="field" :key="field.id"></fieldcanedit-modul>
                </v-card-text>
              </v-card>
            </div>
          </template>
        </template>
        <!-- <pre>{{canvas}}</pre> -->
        <!-- <pre>{{params}}</pre> -->
      </v-container>
    </v-form>

    <v-form v-model="valid" ref="form" v-if="!isCanvas">
      <v-btn
        small
        class="ml-3"
        @click.prevent="submit"
        :class=" { 'primary white--text' : valid, disabled: !valid }"
      >
        <v-icon left>save</v-icon>save
      </v-btn>
      <v-container style="display: grid; justify-items: stretch; grid-gap: 5px 5px;">
        <template v-for="(field, index) in canvasForm.fields">
          <div :key="field.id" :style="'grid-area:' + getGridPosition(field.position) + ''">
            <v-card class="elevation-3" :hover="hover" height="100%">
              <v-card-title>{{field.name}}</v-card-title>
              <v-card-text>
                <field-modul v-bind:index="index" v-bind:fields="field" :key="field.id"></field-modul>
              </v-card-text>
            </v-card>
          </div>
        </template>
        <!-- <pre>{{params}}</pre> -->
      </v-container>
    </v-form>
    <v-container>
      <v-layout>
        <v-flex md6>
          <!-- start comment module-->
          <v-card class="mr-1">
          <base-comment
            v-bind:comments="comments"
            @postComment="postComment"
            @replyComment="replyComment"
          />
          </v-card>
        </v-flex>
        <v-flex md6>
          <!-- start collaborator module-->
          <v-card class="ml-1">
          <base-collaboration
            v-if="collaborators.total != 0"
            v-bind:collaborators="collaborators"
            @removeCollaborator="removeCollaborator"
          />
          </v-card>
          <!-- end collaborator module-->
        </v-flex>
      </v-layout>
    </v-container>

    <business-form v-if="dialogForm" @close="dialogForm = false" @refresh="refresh"></business-form>
  </div>
</template>
<script>
import bus from "@/bus";
import net from "@/config/httpclient";
import notif from "@/config/alerthandling";
import Notification from "@/components/Notification";

import { baseuriMixins } from "@/mixins/baseuriMixins";
import { dashboardMixins } from "@/mixins/dashboardMixins";
import { commentMixins } from "@/mixins/commentMixins";

import FieldModul from "@/components/field/field";
import FieldCanEditModul from "@/components/field/fieldCanEdit";

import BaseCollaboration from "@/components/talent/team/components/BaseCollaboration";
import FormCollaboration from "@/components/talent/team/components/CollaboratorForm";

import BusinessForm from "./BusinessForm";
import BaseComment from "@/components/talent/team/components/BaseComment";

export default {
  mixins: [dashboardMixins, commentMixins, baseuriMixins],
  data: function() {
    return {
      valid: false,
      canvas: { fields: [] },
      canvasName: "",
      canvasForm: "",
      params: {
        formId: "",
        fieldEntries: []
      },
      isCanvas: true,
      isEdit: false,
      loader: false,
      hover: false,
      dialogForm: false,
      selectedDel: false,
      status: {
        success: false,
        error: false,
        info: false,
        warning: false
      },
      err_msg: { details: [""] },
      collaborators: { total: 0, list: [] },
      collaboratorForm: false
    };
  },
  components: {
    BaseCollaboration,
    FormCollaboration,
    BusinessForm,
    BaseComment,
    "notification-alert": Notification,
    "field-modul": FieldModul,
    "fieldcanedit-modul": FieldCanEditModul
  },
  mounted: function() {
    this.getCanvas();
    if (this.checkDashboard) {
      this.loadCollaborator();
    }
    this.loadComment();
  },
  watch: {
    // isCanvas: "getCanvasForm"
  },
  created: function() {
    bus.$on("getValue", (params, index) => {
      this.params.fieldEntries.splice(index, 1, params);
    });
    if (this.checkDashboard) {
      this.comments_uri = this.baseUriTalent.canvas;
    } else {
      this.comments_uri = this.baseUriMentor.canvas;
    }
  },
  methods: {
    submit: function() {
      if (this.$refs.form.validate()) {
        this.addData();
      } else {
        this.$vuetify.goTo(this.$refs.notif, {
          duration: 500,
          offset: 0,
          easing: "linear"
        });
      }
    },
    update: function() {
      if (this.$refs.form2.validate()) {
        this.updateData();
      } else {
        this.$vuetify.goTo(this.$refs.notif, {
          duration: 500,
          offset: 0,
          easing: "linear"
        });
      }
    },
    getGridPosition: function(position) {
      var position = JSON.parse(position);
      return position.grid;
    },
    setFormJSONTemplate: function(data) {
      var array = [];
      for (var i = 0; i < data.fields.length; i++) {
        array.push(new Object({ id: data.fields[i].id, value: "" }));
      }
      this.params.fieldEntries = array;
    },
    getCanvas: function() {
      this.loader = true;
      notif.reset(this);
      var parent_uri = "";
      if (localStorage.getItem("dashboard") == "talent") {
        parent_uri = "/talent/as-team-member/";
      } else if (localStorage.getItem("dashboard") == "mentor") {
        parent_uri =
          "/talent/as-programme-mentor/" +
          this.$route.params.programId +
          "/teams/";
      }
      net
        .getData(
          this,
          parent_uri +
            this.$route.params.teamId +
            "/ideas/" +
            this.$route.params.ideaId +
            "/customer-segments/" +
            this.$route.params.customersegmentId +
            "/personas/" +
            this.$route.params.personaId +
            "/value-propositions/" +
            this.$route.params.valuepropositionId +
            "/business-canvases/" +
            this.$route.params.formId
        )
        .then(res => {
          this.canvas = res.data.data;
          this.canvasName = this.canvas.form.name;
          this.isCanvas = true;
        })
        .catch(error => {
          if (error.status == 404) {
            notif.showInfo(this, error, ["No Canvas Data Found"]);
          } else {
            notif.showError(this, error);
          }
          this.isCanvas = false;
          if (this.checkDashboard) {
            this.getCanvasForm();
          }
        })
        .finally(() => {
          this.loader = false;
        });
    },
    getCanvasForm: function() {
      this.loader = true;
      this.params.formId = this.$route.params.formId;
      net
        .getData(this, "/talent/forms/id/" + this.$route.params.formId)
        .then(res => {
          if (res.data.data) {
            this.canvasForm = res.data.data;
            this.canvasName = this.canvasForm.name;
            this.setFormJSONTemplate(res.data.data);
          } else {
            this.canvasForm = "";
          }
        })
        .catch(error => {
          notif.showError(this, error);
        })
        .finally(() => {
          this.loader = false;
        });
    },
    addData: function() {
      this.params.formId = this.$route.params.formId;
      this.loader = true;
      notif.reset(this);
      net
        .patchData(this, this.baseUriTalent.canvas, this.params)
        .then(res => {
          notif.showSuccess(this, res, ["Data Canvas Saved"]);
          this.isCanvas = true;
          this.isEdit = false;
          this.refresh();
        })
        .catch(error => {
          notif.showError(this, error);
        })
        .finally(() => {
          this.loader = false;
        });
    },
    updateData: function() {
      this.params.formId = this.$route.params.formId;
      this.loader = true;
      notif.reset(this);
      net
        .patchData(this, this.baseUriTalent.canvas, this.params)
        .then(res => {
          notif.showSuccess(this, res, ["Data Canvas Updated"]);
          this.isCanvas = true;
          this.isEdit = false;
          this.getCanvas();
        })
        .catch(error => {
          notif.showError(this, error);
        })
        .finally(() => {
          this.loader = false;
        });
    },
    deleteCanvas: function() {
      this.loader = true;
      net
        .deleteData(
          this,
          this.baseUriTalent.canvas + "/" + +this.$route.params.formId
        )
        .then(res => {
          notif.showInfo(this, res, ["Canvas Data Deleted"]);
          this.getCanvasForm();
          this.canvas = { fields: [] };
          this.isCanvas = false;
        })
        .catch(error => {
          notif.showError(this, error);
        })
        .finally(() => {
          this.loader = false;
          this.selectedDel = false;
        });
    },
    openBusinessForm: function() {
      this.dialogForm = true;
    },
    refresh: function() {
      this.dialogForm = false;
      this.getCanvas();
    },
    loadCollaborator: function() {
      this.loader = true;
      notif.reset(this);
      net
        .getData(
          this,
          this.baseUriTalent.canvas +
            "/" +
            this.$route.params.formId +
            "/collaborators"
        )
        .then(res => {
          if (res.data.data) {
            this.collaborators = res.data.data;
          } else {
            this.collaborators = { total: 0, list: [] };
          }
        })
        .catch(error => {
          notif.showError(this, error);
        })
        .finally(() => {
          this.loader = false;
        });
    },
    openCollaborator: function() {
      this.collaboratorForm = true;
    },
    setUpCollaboration: function(params) {
      net
        .putData(
          this,
          this.baseUriTalent.canvas +
            "/" +
            this.$route.params.formId +
            "/collaborators",
          params
        )
        .then(res => {
          this.collaboratorForm = false;
          this.loadCollaborator();
        })
        .catch(error => {
          notif.showError(this, error);
        })
        .finally(() => {
          this.collaboratorForm = false;
        });
    },
    removeCollaborator: function(mentorId) {
      this.loader = true;
      net
        .deleteData(
          this,
          this.baseUriTalent.canvas +
            "/" +
            this.$route.params.formId +
            "/collaborators/" +
            mentorId
        )
        .then(res => {
          this.loadCollaborator();
        })
        .catch(error => {
          notif.showError(this, error);
        })
        .finally(() => {
          this.loader = false;
        });
    },
    loadComment: function() {
      this.loader = true;
      notif.reset(this);
      net
        .getData(
          this,
          this.comments_uri + "/" + this.$route.params.formId + "/comments"
        )
        .then(res => {
          if (res.data.data) {
            this.comments = res.data.data;
          } else {
            this.comments = { total: 0, list: [] };
          }
        })
        .catch(error => {
          notif.showError(this, error);
        })
        .finally(() => {
          this.loader = false;
        });
    },
    postComment: function(content) {
      net
        .putData(
          this,
          this.comments_uri + "/" + this.$route.params.formId + "/comments",
          content
        )
        .then(res => {
          this.loadComment();
        })
        .catch(error => {
          notif.showError(this, error);
        })
        .finally(() => {});
    },
    replyComment: function(content, id) {
      net
        .putData(
          this,
          this.comments_uri +
            "/" +
            this.$route.params.formId +
            "/comments/" +
            id,
          content
        )
        .then(res => {
          this.loadComment();
        })
        .catch(error => {
          notif.showError(this, error);
        })
        .finally(() => {});
    }
  }
};
</script>

<style scoped>
.lain {
  background: #fb8c00;
  width: 35px;
  height: 4px;
  margin-left: 17px;
}
</style>
