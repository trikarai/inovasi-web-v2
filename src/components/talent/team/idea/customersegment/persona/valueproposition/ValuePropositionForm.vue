<template>
  <transition name="modal">
    <div class="modal-mask">
      <div class="modal-wrapper" @click="$emit('close')">
        <div class="modal-container" @click.stop>
          <notification-alert ref="notif" v-bind:err_msg="err_msg" v-bind:status="status" />
          <v-card elevation="0" width="450" style="padding:0px 30px 20px 30px;margin-top:8px">
            <v-card class="taitel primary white--text elevation-5">
              <h3
                v-if="edit"
                class="headline mb-0 font-weight-light"
              >{{ $vuetify.lang.t('$vuetify.action.edit') }} Value Proposition</h3>
              <h3
                v-if="!edit"
                class="headline mb-0 font-weight-light"
              >{{ $vuetify.lang.t('$vuetify.action.add') }} Value Proposition</h3>
            </v-card>
            <v-card-text class="pt-4">
              <div>
                <v-form v-model="valid" ref="form">
                  <v-textarea
                    label="Value Proposition"
                    v-model="params.description"
                    :rules="nameRules"
                    counter
                    required
                    placeholder="Value Proposition adalah sebuah pernyataan terkait value yang akan diperoleh oleh pelanggan dengan menggunakan produk/jasa anda. Isinya dapat berupa manfaat-manfaat yang akan diperoleh pelanggan. Selain menjelaskan manfaat, Value Proposition yang baik juga menjelaskan mengapa produk/jasa anda lebih baik dibandingkan dengan kompetitor."
                  ></v-textarea>
                  <!-- hint="Value Proposition adalah sebuah pernyataan terkait value yang akan diperoleh oleh pelanggan dengan menggunakan produk/jasa anda. Isinya dapat berupa manfaat-manfaat yang akan diperoleh pelanggan. Selain menjelaskan manfaat, Value Proposition yang baik juga menjelaskan mengapa produk/jasa anda lebih baik dibandingkan dengan kompetitor." -->

                  <v-textarea
                    label="Products And Services"
                    hint
                    v-model="params.productsAndServices"
                    counter
                    required
                  ></v-textarea>
                  <v-textarea
                    label="Gain Creators"
                    hint
                    v-model="params.gainCreators"
                    counter
                    required
                  ></v-textarea>
                  <v-textarea
                    label="Pain Relievers"
                    hint
                    v-model="params.painRelievers"
                    counter
                    required
                  ></v-textarea>
                  <v-textarea label="Gains" hint v-model="params.gains" counter required></v-textarea>
                  <v-textarea label="Pains" hint v-model="params.pains" counter required></v-textarea>
                  <v-textarea
                    label="Customer Jobs"
                    hint
                    v-model="params.customerJobs"
                    counter
                    required
                  ></v-textarea>

                  <v-layout justify-space-between>
                    <v-btn
                      v-if="edit == false"
                      @click.prevent="submit"
                      class="mt-5"
                      :class=" { 'primary white--text' : valid}"
                      :disabled="!valid"
                      block
                    >{{ $vuetify.lang.t('$vuetify.action.add')}}</v-btn>

                    <v-btn
                      v-else
                      @click="update"
                      class="mt-5"
                      :class=" { 'primary white--text' : valid}"
                      :disabled="!valid"
                      block
                    >{{ $vuetify.lang.t('$vuetify.action.update')}}</v-btn>

                    <loader-dialog v-model="loader"></loader-dialog>
                  </v-layout>
                </v-form>
              </div>
            </v-card-text>
          </v-card>
        </div>
      </div>
    </div>
  </transition>
</template>
<script>
import net from "@/config/httpclient";
import notif from "@/config/alerthandling";
import notification from "@/components/Notification";

export default {
  props: ["id", "edit", "view", "data"],
  data: function() {
    return {
      valid: false,
      loader: false,
      status: {
        success: false,
        error: false,
        info: false,
        warning: false
      },
      err_msg: { code: 666, type: "", details: [""] },
      error: "error",
      params: {
        description: "",
        customerJobs: "",
        pains: "",
        gains: "",
        productsAndServices: "",
        gainCreators: "",
        painRelievers: ""
      },
      nameRules: [
        v => !!v || "Description is required",
        v => v.length >= 3 || "Description must be more than 3 characters"
      ]
    };
  },
  components: {
    "notification-alert": notification
  },
  created: function() {},
  mounted: function() {
    if (this.edit) {
      this.getSingleData();
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
      if (this.$refs.form.validate()) {
        this.updateData();
      } else {
        this.$vuetify.goTo(this.$refs.notif, {
          duration: 500,
          offset: 0,
          easing: "linear"
        });
      }
    },
    getSingleData: function() {
      net
        .getData(
          this,
          "/talent/as-team-member/" +
            this.$route.params.teamId +
            "/ideas/" +
            this.$route.params.ideaId +
            "/customer-segments/" +
            this.$route.params.customersegmentId +
            "/personas/" +
            this.$route.params.personaId +
            "/value-propositions/" +
            this.data.id
        )
        .then(res => {
          this.params = res.data.data;
        })
        .catch(error => {
          notif.showError(this, error);
        })
        .finally(() => {
          this.loader = false;
        });
    },
    addData: function() {
      this.loader = true;
      this.status.error = false;
      net
        .postData(
          this,
          "/talent/as-team-member/" +
            this.$route.params.teamId +
            "/ideas/" +
            this.$route.params.ideaId +
            "/customer-segments/" +
            this.$route.params.customersegmentId +
            "/personas/" +
            this.$route.params.personaId +
            "/value-propositions",
          this.params
        )
        .then(res => {
          this.$emit("refresh");
        })
        .catch(error => {
          notif.showError(this, error);
        })
        .finally(() => {
          this.loader = false;
        });
    },
    updateData: function() {
      var app = this;
      this.loader = true;
      this.status.error = false;
      net
        .putData(
          this,
          "/talent/as-team-member/" +
            this.$route.params.teamId +
            "/ideas/" +
            this.$route.params.ideaId +
            "/customer-segments/" +
            this.$route.params.customersegmentId +
            "/personas/" +
            this.$route.params.personaId +
            "/value-propositions/" +
            this.data.id,
          this.params
        )
        .then(res => {
          this.$emit("refresh");
        })
        .catch(error => {
          notif.showError(this, error);
        })
        .finally(() => {
          this.loader = false;
        });
    }
  }
};
</script>
<style scoped>
@import "../../../../../../css/modal.css";

.taitel {
  padding: 24px;
  width: 90%;
  margin: 0 auto;
  bottom: 27px;
  z-index: 2;
}
.modal-mask {
  /* display: inherit !important; */
}
</style>
