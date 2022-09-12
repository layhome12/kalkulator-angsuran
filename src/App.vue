<template>
  <div class="content-main">
    <div class="row row-main justify-content-center align-items-center">
      <div class="col-lg-9 col-md-10 col-12">
        <div class="card">
          <div class="card-body">
            <div class="row">
              <div class="col-lg-4">
                <form @submit.prevent="onSubmitted">
                  <b-form-group label="Nominal">
                    <b-input-group size="md" prepend="$">
                      <money3
                        v-model="nominal"
                        v-bind="config"
                        class="form-control"
                      ></money3>
                    </b-input-group>
                    <small v-if="nominalCheck.isError" class="text-danger">{{
                      nominalCheck.msgError
                    }}</small>
                  </b-form-group>
                  <b-form-group label="DP">
                    <b-input-group size="md" prepend="$">
                      <money3
                        v-model="dp"
                        v-bind="config"
                        class="form-control"
                      ></money3>
                    </b-input-group>
                    <small v-if="dpCheck.isError" class="text-danger">{{
                      dpCheck.msgError
                    }}</small>
                  </b-form-group>
                  <div class="row">
                    <div class="col-6">
                      <b-form-group label="Waktu">
                        <b-input-group size="md" append="Bulan">
                          <b-form-input
                            v-model="bulan"
                            type="number"
                            min="1"
                            placeholder="12"
                            required
                          ></b-form-input>
                        </b-input-group>
                      </b-form-group>
                    </div>
                    <div class="col-6">
                      <b-form-group label="Persen">
                        <b-input-group size="md" append="%">
                          <b-form-input
                            v-model="persen"
                            @input="
                              (val) => {
                                if (val) {
                                  persen = parseFloat(val) / 100;
                                } else {
                                  persen = ``;
                                }
                              }
                            "
                            type="number"
                            min="1"
                            max="100"
                            placeholder="100"
                            step="0.1"
                            required
                          ></b-form-input>
                        </b-input-group>
                      </b-form-group>
                    </div>
                  </div>
                  <div class="row">
                    <div class="col-6">
                      <b-button
                        type="submit"
                        class="btn-block"
                        variant="primary"
                        >Hitung</b-button
                      >
                    </div>
                    <div class="col-6">
                      <b-button
                        @click="resetValues()"
                        class="btn-block"
                        variant="warning"
                        >Reset</b-button
                      >
                    </div>
                  </div>
                </form>
              </div>
              <div class="col-lg-8">
                <div class="card full-height mt-md-0 mt-3">
                  <div class="card-body">
                    <div class="row full-height p-2">
                      <div class="col-lg-6 col-12 mt-2 mb-2">
                        <div class="card">
                          <div class="card-header">Angsuran Perbulan</div>
                          <div class="card-body">
                            <h4>Rp. {{ angsuran }}</h4>
                          </div>
                        </div>
                      </div>
                      <div class="col-lg-6 col-12 mt-2 mb-2">
                        <div class="card">
                          <div class="card-header">Keuntungan Perbulan</div>
                          <div class="card-body">
                            <h4>Rp. {{ marginMonth }}</h4>
                          </div>
                        </div>
                      </div>
                      <div class="col-lg-6 col-12 mt-2 mb-2">
                        <div class="card">
                          <div class="card-header">Margin / Keuntungan</div>
                          <div class="card-body">
                            <h4>Rp. {{ margin }}</h4>
                          </div>
                        </div>
                      </div>
                      <div class="col-lg-6 col-12 mt-2 mb-2">
                        <div class="card">
                          <div class="card-header">Total Nominal</div>
                          <div class="card-body">
                            <h4>Rp. {{ totalNominal }}</h4>
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { Money3Component } from "v-money3";

export default {
  name: "App",
  components: { money3: Money3Component },
  data() {
    return {
      nominal: "",
      dp: 0,
      bulan: "",
      persen: "",
      angsuran: "-",
      margin: "-",
      marginMonth: "-",
      totalNominal: "-",

      //Masking
      config: {
        masked: false,
        prefix: "",
        suffix: "",
        thousands: ",",
        precision: 0,
        allowBlank: true,
        disableNegative: true,
      },

      //isError
      nominalCheck: {
        isError: false,
        msgError: "",
      },
      dpCheck: {
        isError: false,
        msgError: "",
      },
    };
  },
  methods: {
    onSubmitted() {
      let validation = this.isChecking({ reset: false });
      if (!validation) return;

      let hNominal = this.nominal - this.dp;
      let hPersen = hNominal * this.persen;
      let hMargin = hPersen * this.bulan;
      let hTotalNomial = parseInt(hNominal) + parseInt(hMargin);
      let hAngsuran = Math.ceil(hTotalNomial / this.bulan);

      this.resultValue(hAngsuran, hMargin, hPersen, hTotalNomial);
      this.isChecking({ reset: true });
    },
    isChecking({ reset = false }) {
      let resultValidation = true;

      if (this.nominal == 0 || this.nominal == "") {
        this.nominalCheck.isError = true;
        this.nominalCheck.msgError = "Nominal Masih Kosong";
        resultValidation = false;
      } else {
        this.nominalCheck.isError = false;
        this.nominalCheck.msgError = "";
      }

      if (parseInt(this.dp) >= parseInt(this.nominal)) {
        this.dpCheck.isError = true;
        this.dpCheck.msgError = "DP Harus Kurang Dari Nominal";
        resultValidation = false;
      } else {
        this.dpCheck.isError = false;
        this.dpCheck.msgError = "";
      }

      if (reset) {
        this.nominalCheck.isError = false;
        this.nominalCheck.msgError = "";
        this.dpCheck.isError = false;
        this.dpCheck.msgError = "";
      }

      return resultValidation;
    },
    resultValue(hAngsuran, hMargin, hPersen, hTotalNomial) {
      this.angsuran = hAngsuran
        .toFixed(2)
        .replace(".", ",")
        .toString()
        .replace(/\B(?=(\d{3})+(?!\d))/g, ",");
      this.margin = hMargin
        .toFixed(2)
        .replace(".", ",")
        .toString()
        .replace(/\B(?=(\d{3})+(?!\d))/g, ",");
      this.marginMonth = hPersen
        .toFixed(2)
        .replace(".", ",")
        .toString()
        .replace(/\B(?=(\d{3})+(?!\d))/g, ",");
      this.totalNominal = hTotalNomial
        .toFixed(2)
        .replace(".", ",")
        .toString()
        .replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    },
    resetValues() {
      this.nominal = "";
      this.dp = 0;
      this.bulan = "";
      this.persen = "";
      this.angsuran = "-";
      this.margin = "-";
      this.marginMonth = "-";
      this.totalNominal = "-";
    },
  },
};
</script>

<style>
.content-main,
.content-main > .row-main {
  height: 100vh;
  margin: 0;
}
.full-height {
  height: 100% !important;
}
.btn-block {
  width: 100%;
}
</style>
