<template>
  <transition name="slide" mode="out-in">
    <div>
      <v-container>
        <notification-alert ref="notif" v-bind:err_msg="err_msg" v-bind:status="status" />

        <v-dialog v-model="loader2" hide-overlay persistent width="300">
          <v-card color="primary">
            <v-card-text>
              {{ $vuetify.lang.t('$vuetify.info.standby') }}
              <v-progress-linear indeterminate color="white" class="mb-0"></v-progress-linear>
            </v-card-text>
          </v-card>
        </v-dialog>
        <!-- {{participants.list}} -->
        <v-data-table
          :loading="loader"
          :headers="headers"
          :items="participants.list"
          class="elevation-1"
        >
          <template v-slot:item.status="{item}">

            <v-chip :color="setColor(item.status)" text-color="white">
              <v-avatar left>
                <v-icon>{{setIcon(item.status)}}</v-icon>
              </v-avatar>
              {{ item.status }}
            </v-chip>
          </template>
          <template v-slot:item.action="{item}">
            <v-btn rounded small @click.stop="openDetail(item.id)" class="ma-1">
              <v-icon small>search</v-icon>
            </v-btn>
            <v-btn v-if="item.status == 'applied'" small color="green" @click="acceptAct(item.id)" class="ma-1">
              <v-icon small>check_circle_outline</v-icon>Accept
            </v-btn>
            <v-btn
              v-if="item.status == 'applied'"
              small
              color="warning"
              @click="rejectAct(item.id)"
            >
              <v-icon small>highlight_off</v-icon>Reject
            </v-btn>
            <v-btn v-if="item.status == 'active'" small color="red" @click="expellAct(item.id)">
              <v-icon small>block</v-icon>Expell
            </v-btn>
            

            <v-flex shrink>
              <v-expand-transition>
                <div v-show="item.id == selectedIndexAcc" style="white-space: nowrap">
                  Confirm to Accept!
                  <v-btn color="green" @click="acceptParticipant(item.id)" class="ma-2">
                    <v-icon>check</v-icon>
                  </v-btn>
                  <v-btn color="red" @click="selectedIndexAcc = null">
                    <v-icon>close</v-icon>
                  </v-btn>
                </div>
              </v-expand-transition>
            </v-flex>

            <v-flex shrink>
              <v-expand-transition>
                <div v-show="item.id == selectedIndexRej" style="white-space: nowrap">
                  Confirm to Reject!
                  <v-btn color="green" @click="rejectParticipant(item.id)" class="ma-2">
                    <v-icon>check</v-icon>
                  </v-btn>
                  <v-btn color="red" @click="selectedIndexRej = null">
                    <v-icon>close</v-icon>
                  </v-btn>
                </div>
              </v-expand-transition>
            </v-flex>

            <v-flex shrink>
              <v-expand-transition>
                <div v-show="item.id == selectedIndexExp" style="white-space: nowrap">
                  Confirm to Expell!
                  <v-btn color="green" @click="expellParticipant(item.id)" class="ma-2">
                    <v-icon>check</v-icon>
                  </v-btn>
                  <v-btn color="red" @click="selectedIndexExp = null">
                    <v-icon>close</v-icon>
                  </v-btn>
                </div>
              </v-expand-transition>
            </v-flex>
          </template>
        </v-data-table>
      </v-container>
      <v-container>
        <v-layout>
          <v-dialog v-model="dialog" max-width="500">
            <v-card v-if="loaderDialog">
              <v-card-text>
                <v-progress-linear :indeterminate="loaderDialog"></v-progress-linear>
              </v-card-text>
            </v-card>
            <v-card v-else>
              <v-card-title class="headline" primary-title>{{singleData.team.name}}</v-card-title>
              <v-card-text>
                Status : {{singleData.status}}
                <br />
                Registration Time : {{singleData.registrationTime}}
                <br />
              </v-card-text>
              <v-divider></v-divider>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="red" text @click="dialog = false">
                  <v-icon>close</v-icon>Close
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-layout>
      </v-container>
    </div>
  </transition>
</template>
<script>
import auth from "@/config/auth";
import net from "@/config/httpclient";
import notif from "@/config/alerthandling";
import Notification from "@/components/Notification";
export default {
  components: {
    "notification-alert": Notification
  },
  data() {
    return {
      expand: false,
      expand2: false,
      expand3: false,
      user: "",
      participantId: "",
      status: {
        success: false,
        error: false,
        info: false,
        warning: false
      },
      singleData: {
        id: "",
        status: "",
        registrationTime: "",
        team: {
          id: "",
          name: ""
        }
      },
      err_msg: { details: [""] },
      loader: false,
      loader2: false,
      loaderDialog: false,
      dialogDel: false,
      dialog: false,
      edit: false,
      view: false,
      selectedIndex: null,
      selectedIndexAcc: null,
      selectedIndexRej: null,
      selectedIndexExp: null,
      headers: [
        {
          text: "Participant",
          align: "left",
          sortable: false,
          value: "team.name"
        },
        { text: "Status", value: "status", sortable: false },
        { text: "Actions", value: "action", sortable: false, align: "right" }
      ],
      participants: {
        total: 0,
        list: []
      }
    };
  },
  created: function() {},
  mounted: function() {
    this.getDataList();
  },
  methods: {
    setColor: function(status) {
      var color = "grey";
      if (status === "active") {
        color = "green";
      } else if (status === "applied") {
        color = "blue";
      } else if (status === "quit") {
        color = "red";
      } else if (status === "cancelled") {
        color = "warning";
      } else if (status === "expelled") {
        color = "red";
      } else {
      }
      return color;
    },
    setIcon: function(status) {
      var icon = "check";
      if (status === "active") {
        icon = "check_circle";
      } else if (status === "applied") {
        icon = "hourglass_empty";
      } else if (status === "quit") {
        icon = "flag";
      } else if (status === "cancelled") {
        icon = "cancel";
      } else if (status === "expelled") {
        icon = "remove_circle";
      } else {
      }
      return icon;
    },
    getDataList: function() {
      this.loader = true;
      notif.reset(this);
      net
        .getData(
          this,
          "/talent/as-programme-coordinator/" +
            this.$route.params.programId +
            "/participants"
        )
        .then(res => {
          if (res.data.data) {
            this.participants = res.data.data;
          } else {
            this.participants.list = [];
          }
        })
        .catch(error => {
          notif.showError(this, error);
        })
        .finally(() => {
          this.loader = false;
        });
    },
    getSingleData: function(id) {
      notif.reset(this);
      this.loaderDialog = true;
      net
        .getData(
          this,
          "/talent/as-programme-coordinator/" +
            this.$route.params.programId +
            "/participants/" +
            id
        )
        .then(res => {
          if (res.data.data) {
            this.singleData = res.data.data;
          } else {
            this.singleData = {};
          }
        })
        .catch(error => {
          notif.showError(this, error);
          this.dialog = false;
        })
        .finally(() => {
          this.loaderDialog = false;
        });
    },
    openDetail: function(id) {
      this.participantId = id;
      this.dialog = true;
      this.loaderDialog = true;
      this.getSingleData(id);
    },
    acceptAct: function(index) {
      this.selectedIndexRej = null;
      this.selectedIndexExp = null;
      if (this.selectedIndexAcc == index) {
        this.selectedIndexAcc = null;
      } else {
        this.selectedIndexAcc = index;
      }
    },
    acceptParticipant: function(id) {
      this.loader2 = true;
      notif.reset(this);
      net
        .putData(
          this,
          "/talent/as-programme-coordinator/" +
            this.$route.params.programId +
            "/participants/" +
            id +
            "/accept"
        )
        .then(res => {
          this.refresh();
        })
        .catch(error => {
          notif.showError(this, error);
        })
        .finally(() => {
          this.selectedIndexAcc = null;
          this.loader2 = false;
          this.refresh();
        });
    },
    rejectAct: function(index) {
      this.selectedIndexAcc = null;
      this.selectedIndexExp = null;
      if (this.selectedIndexRej == index) {
        this.selectedIndexRej = null;
      } else {
        this.selectedIndexRej = index;
      }
    },
    rejectParticipant: function(id) {
      this.loader2 = true;
      notif.reset(this);
      net
        .putData(
          this,
          "/talent/as-programme-coordinator/" +
            this.$route.params.programId +
            "/participants/" +
            id +
            "/reject"
        )
        .then(res => {
          this.refresh();
        })
        .catch(error => {
          notif.showError(this, error);
        })
        .finally(() => {
          this.selectedIndexRej = null;
          this.loader2 = false;
        });
    },
    expellAct: function(index) {
      this.selectedIndexRej = null;
      this.selectedIndexAcc = null;
      if (this.selectedIndexExp == index) {
        this.selectedIndexExp = null;
      } else {
        this.selectedIndexExp = index;
      }
    },
    expellParticipant: function(id) {
      this.loader = true;
      notif.reset(this);
      net
        .putData(
          this,
          "/talent/as-programme-coordinator/" +
            this.$route.params.programId +
            "/participants/" +
            id +
            "/expell"
        )
        .then(res => {
          this.refresh();
        })
        .catch(error => {
          notif.showError(this, error);
        })
        .finally(() => {
          this.selectedIndexExp = null;
          this.loader = false;
        });
    },
    refresh: function() {
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