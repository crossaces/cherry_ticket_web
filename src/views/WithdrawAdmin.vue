<template>
  <v-main class="bg-background">
    <v-card data-aos="zoom-in" class="m-10 rounded-lg">
      <v-card-title>
        <v-text-field
          dense
          v-model="search"
          color="cherryv"
          append-icon="mdi-magnify"
          label="Search"
          single-line
          hide-details
        >
        </v-text-field>

        <v-spacer></v-spacer>
        <v-btn color="indigov" class="whitev--text" @click="exportshow()"
          ><v-icon left>mdi-note</v-icon>Export Gained History</v-btn
        >
      </v-card-title>
      <v-data-table :headers="headers" :items="data" :search="search">
        <template v-slot:[`item.EO`]="{ item }">
          {{ item.eo.NAMA_EO }}
        </template>
        <template v-slot:[`item.STATUS_WITHDRAW`]="{ item }">
          <v-chip outlined :color="warnacheck(item.STATUS_WITHDRAW)">{{
            item.STATUS_WITHDRAW
          }}</v-chip>
        </template>
        <template v-slot:[`item.TOTAL_WITHDRAW`]="{ item }">
          Rp. {{ formatPrice(item.TOTAL_WITHDRAW) }}
        </template>
        <template v-slot:[`item.TGL_WITHDRAW`]="{ item }">
          {{ formatTanggal(item.TGL_WITHDRAW) }}
        </template>
        <template v-slot:[`item.actions`]="{ item }">
          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <v-btn
                icon
                small
                class="mr-2"
                v-bind="attrs"
                v-on="on"
                @click="
                  item.STATUS_WITHDRAW == 'Sended'
                    ? pending(item.ID_WITHDRAW)
                    : sended(item.ID_WITHDRAW)
                "
              >
                <v-icon
                  :color="
                    item.STATUS_WITHDRAW == 'Sended' ? 'dangerv' : 'success'
                  "
                  >mdi-brightness-1</v-icon
                >
              </v-btn>
            </template>
            <span v-if="item.STATUS_WITHDRAW == 'Sended'"> Pending</span>
            <span v-if="item.STATUS_WITHDRAW != 'Sended'"> Sended</span>
          </v-tooltip>
        </template>
      </v-data-table>
    </v-card>
    <v-dialog v-model="dialogexport" max-width="1000px">
      <v-card>
        <v-toolbar color="indigov text-lg font-bold" dark>
          {{ judul }}
        </v-toolbar>
        <v-card-text>
          <v-form ref="export" lazy-validation>
            <v-row class="mt-10">
              <p class="text-indigo text-xl font-semibold opacity-80 px-3">
                Choose range for get export withdraw history:
              </p>
            </v-row>
            <v-row class="px-3">
              <v-text-field
                label="Start Date ~ End Date"
                :value="dateRangeText"
                placeholder="Start Date ~ End Date"
                outlined
                :rules="daterules"
                readonly
                color="indigov"
              ></v-text-field>
            </v-row>
            <v-row class="px-3">
              <v-date-picker
                scrollable
                dark
                header-color="indigov"
                v-model="datenews"
                no-title
                :show-current="false"
                color="cherryv"
                @change="dateinit(datenews)"
                full-width
                range
              ></v-date-picker>
            </v-row>
          </v-form>
        </v-card-text>
        <v-card-actions class="justify-end">
          <v-btn color="indigov" class="whitev--text" @click="tutup()"
            >Close</v-btn
          >
          <v-btn class="whitev--text" color="cherryv" @click="exportwithdraw()"
            >Export</v-btn
          >
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-dialog v-model="dialoghapus" persistent max-width="600px">
      <v-card>
        <v-toolbar color="indigov text-lg font-bold" dark>{{
          judul
        }}</v-toolbar>
        <v-card-text>
          <div class="w-full h-full flex items-center p-10">
            <div class="m-auto">
              <v-icon color="dangerv" class="w-full my-3" size="80px">
                mdi-alert-circle-outline</v-icon
              >
              <h2 class="text-indigo text-3xl mt-3 font-bold">
                Are You Sure Delete This Data?
              </h2>
            </div>
          </div>
        </v-card-text>
        <v-card-actions class="justify-end">
          <v-btn
            height="30px"
            color="indigov"
            class="whitev--text"
            @click="dialoghapus = false"
            >Close</v-btn
          >
          <v-btn
            height="30px"
            class="whitev--text"
            color="cherryv"
            @click="hapus()"
            >Delete</v-btn
          >
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-dialog v-model="dialoggambar" persistent max-width="600px">
      <v-card>
        <v-toolbar color="indigov text-lg font-bold" dark
          >Picture withdraw</v-toolbar
        >
        <v-card-text>
          <v-img :src="image" class="mx-auto mt-10" max-width="400px"></v-img>
        </v-card-text>
        <v-card-actions class="justify-end">
          <v-btn
            color="indigov"
            class="whitev--text"
            @click="dialoggambar = false"
            >Close</v-btn
          >
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-dialog v-model="dialogform" max-width="1000px">
      <v-card>
        <v-toolbar color="indigov text-lg font-bold" dark>
          {{ judul }}
        </v-toolbar>
        <v-card-text>
          <v-form ref="form" lazy-validation>
            <v-row class="mt-10">
              <v-col sm="6">
                <v-autocomplete
                  v-model="form.METHOD_PAYMENT"
                  :items="status"
                  item-text="nama"
                  item-value="value"
                  color="indigov"
                  item-color="cherryv"
                  :rules="rules"
                  label="Method Payment"
                  required
                  outlined
                >
                </v-autocomplete>
              </v-col>
              <v-col sm="6">
                <v-text-field
                  label="Withdraw"
                  v-model="form.JUMLAH_WITHDRAW"
                  :rules="withdrawrules"
                  placeholder="Withdraw"
                  outlined
                  color="indigov"
                  type="number"
                  required
                ></v-text-field>
              </v-col>
            </v-row>
            <v-row>
              <v-col sm="6">
                <v-text-field
                  label="Account Name"
                  v-model="form.NAMA_TUJUAN"
                  :rules="rules"
                  placeholder="Account Name"
                  outlined
                  color="indigov"
                  required
                ></v-text-field>
              </v-col>
              <v-col>
                <v-text-field
                  label="Account Number"
                  v-model="form.NOMOR_TRANSAKSI"
                  :rules="withdrawrules"
                  placeholder="Account Number"
                  outlined
                  color="indigov"
                  type="number"
                  required
                ></v-text-field>
              </v-col>
            </v-row>
          </v-form>
          <div class="relative max-w-xl py-2 text-gray-700">
            <div class="flex flex-col h-full">
              <div class="my-auto">
                <div class="text-indigo text-md opawithdraw-80">
                  Total Withdraw + 5% Admin
                </div>
                <div class="text-indigo text-md font-bold">
                  Rp.
                  {{
                    form.JUMLAH_WITHDRAW == null
                      ? "0"
                      : formatPrice(
                          String(
                            Math.ceil(
                              form.JUMLAH_WITHDRAW -
                                (form.JUMLAH_WITHDRAW * 5) / 100
                            )
                          )
                        )
                  }}
                </div>
              </div>
            </div>
          </div>
        </v-card-text>
        <v-card-actions class="justify-end">
          <v-btn color="indigov" class="whitev--text" @click="tutup()"
            >Close</v-btn
          >
          <v-btn
            v-if="cekaction == 'tambah'"
            class="whitev--text"
            color="cherryv"
            @click="tambah()"
            >Add</v-btn
          >
          <v-btn
            v-if="cekaction == 'edit'"
            class="whitev--text"
            color="cherryv"
            @click="edit()"
            >Edit</v-btn
          >
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-snackbar v-model="snackbar" :color="color" timeout="2000" bottom>
      {{ error_message }}
    </v-snackbar>
  </v-main>
</template>

<script>
export default {
  data() {
    return {
      datenews: [],
      momentdate: [],
      judul: "",
      income: 0,
      cekaction: false,
      image: "",
      error_message: "",
      status: [
        {
          nama: "BCA Transfer",
          value: "BCA Transfer",
        },
        {
          nama: "Dana",
          value: "Dana",
        },
      ],
      rules: [(v) => !!v || "This field is required"],
      withdrawrules: [
        (v) => !!v || "This field is required",
        (v) =>
          (!!v && parseInt(this.income) >= this.form.JUMLAH_WITHDRAW) ||
          "You balance not enough",
      ],
      daterules: [
        (v) => !!v || "This field is required",
        (v) => (!!v && this.datenews[1] != null) || "Select one more date",
      ],
      show1: false,
      show2: false,
      snackbar: false,
      color: "",
      search: null,
      total_withdraw: 0,
      uploadimage: null,
      dilaog: false,
      dialogexport: false,
      dialoghapus: false,
      gambar: null,
      data: [],
      withdraw: new FormData(),
      dialogform: false,
      dialoggambar: false,
      id_data: null,
      form: {
        JUMLAH_WITHDRAW: null,
        NOMOR_TRANSAKSI: null,
        NAMA_TUJUAN: null,
        METHOD_PAYMENT: null,
      },
      headers: [
        {
          text: "Event Organizer",
          value: "EO",
        },
        {
          text: "Account Name",
          value: "NAMA_TUJUAN",
        },
        { text: "Date Withdraw", value: "TGL_WITHDRAW" },
        { text: "Total Withdraw", value: "TOTAL_WITHDRAW" },
        { text: "Method Payment", value: "METHOD_PAYMENT" },
        { text: "Account Number", value: "NOMOR_TRANSAKSI" },
        { text: "Status Withdraw", value: "STATUS_WITHDRAW" },
        { text: "Actions", value: "actions" },
      ],
    };
  },
  methods: {
    exportwithdraw() {
      if (this.$refs.export.validate()) {
        var url =
          this.$api +
          "/laporanwithdrawadmin/" +
          this.datenews[0] +
          "/" +
          this.datenews[1];
        this.$http
          .get(url, {
            headers: {
              Authorization: "Bearer " + localStorage.getItem("token"),
            },
            responseType: "blob",
          })
          .then((response) => {
            var bambang =
              this.$api +
              "/laporanwithdrawadmin/" +
              this.datenews[0] +
              "/" +
              this.datenews[1];
            window.open(bambang);
            const url = window.URL.createObjectURL(new Blob([response.data]));
            const link = document.createElement("a");
            link.href = url;
            document.body.appendChild(link);
          })
          .catch((error) => {
            console.log(error.response.data.message);
          });
      }
    },
    sended(id) {
      var url = this.$api + "/withdraw/" + id;
      let newData = {
        status: "Sended",
      };
      this.$http
        .put(url, newData, {
          headers: {
            Authorization: "Bearer " + localStorage.getItem("token"),
          },
        })
        .then((response) => {
          this.error_message = response.data.message;
          this.color = "success";
          this.snackbar = true;
          this.readData();
        })
        .catch((error) => {
          this.error_message = error.response.data.message;
          this.color = "dangerv";
          this.snackbar = true;
        });
    },
    exportshow() {
      this.dialogexport = true;
      this.judul = "Gained History";
    },
    pending(id) {
      var url = this.$api + "/withdraw/" + id;
      let newData = {
        status: "Pending",
      };
      this.$http
        .put(url, newData, {
          headers: {
            Authorization: "Bearer " + localStorage.getItem("token"),
          },
        })
        .then((response) => {
          this.error_message = response.data.message;
          this.color = "success";
          this.snackbar = true;
          this.readData();
        })
        .catch((error) => {
          this.error_message = error.response.data.message;
          this.color = "dangerv";
          this.snackbar = true;
        });
    },
    tambah() {
      if (this.$refs.form.validate()) {
        var url = this.$api + "/withdraw";
        this.withdraw.append("jumlah_withdraw", this.form.JUMLAH_WITHDRAW),
          this.withdraw.append("nomor_transaksi", this.form.NOMOR_TRANSAKSI),
          this.withdraw.append(
            "income_admin",
            Math.ceil((this.form.JUMLAH_WITHDRAW * 5) / 100)
          ),
          this.withdraw.append(
            "total_withdraw",
            this.form.JUMLAH_WITHDRAW - (this.form.JUMLAH_WITHDRAW * 5) / 100
          ),
          this.withdraw.append("ideo", localStorage.getItem("ideo")),
          this.withdraw.append("method_payment", this.form.METHOD_PAYMENT),
          this.withdraw.append("nama_tujuan", this.form.NAMA_TUJUAN),
          this.$http
            .post(url, this.withdraw, {
              headers: {
                Authorization: "Bearer " + localStorage.getItem("token"),
              },
            })
            .then((response) => {
              this.error_message = response.data.message;
              this.color = "success";
              this.snackbar = true;
              this.dialogform = false;
              this.tutup();
              this.readData();
            })
            .catch((error) => {
              this.error_message = error.response.data.message;
              if (error.response.data.message.nama_kota != null) {
                this.error_message = "The withdraw name already taken";
              }
              if (error.response.data.message.gambar_kota != null) {
                this.error_message = "Image must be field";
              }
              this.color = "dangerv";
              this.snackbar = true;
            });
      }
    },
    showgambar(item) {
      this.dialoggambar = true;
      this.image = this.$image + "/GambarKota/" + item;
    },
    tutup() {
      this.dialogform = false;
      this.form = [];
      this.datenews = [];
      this.momentdate = [];
      this.show1 = false;
      this.change = false;
      this.show2 = false;
      this.dialogexport = false;
      this.$refs.form.reset();
      this.$refs.export.reset();
      this.uploadimage = null;
    },
    showdialogh(id) {
      this.dialoghapus = true;
      this.judul = "Delete withdraw";
      this.id_data = id;
    },
    showdialogt() {
      this.dialogform = true;
      this.judul = "Add withdraw";
      this.cekaction = "tambah";
    },
    showdialoge(item) {
      this.dialogform = true;
      this.judul = "Edit withdraw";
      this.cekaction = "edit";
      this.id_data = item.ID_KOTA;
      var url = this.$api + "/kota/" + this.id_data;
      this.$http
        .get(url, {
          headers: {
            Authorization: "Bearer " + localStorage.getItem("token"),
          },
        })
        .then((response) => {
          this.form.NAMA_KOTA = response.data.data.NAMA_KOTA;
          this.uploadimage =
            this.$image + "/GambarKota/" + response.data.data.GAMBAR_KOTA;
        });
    },
    readData() {
      var url1 = this.$api + "/dashboard/" + localStorage.getItem("ideo");
      this.$http
        .get(url1, {
          headers: {
            Authorization: "Bearer " + localStorage.getItem("token"),
          },
        })
        .then((response) => {
          this.income = response.data.data.sisa_saldo;
        });

      var url = this.$api + "/withdraw";
      this.$http
        .get(url, {
          headers: {
            Authorization: "Bearer " + localStorage.getItem("token"),
          },
        })
        .then((response) => {
          this.data = response.data.data;
        });
    },
    dateinit(value) {
      if (this.$moment(value[1]).isBefore(value[0])) {
        var temp;
        temp = value[1];
        this.datenews[1] = this.datenews[0];
        this.datenews[0] = temp;
        this.momentdate = [
          this.$moment(this.datenews[0]).format("dddd, MMMM D YYYY"),
          this.$moment(this.datenews[1]).format("dddd, MMMM D YYYY"),
        ];
      } else {
        this.momentdate = [
          this.$moment(this.datenews[0]).format("dddd, MMMM D YYYY"),
          this.$moment(this.datenews[1]).format("dddd, MMMM D YYYY"),
        ];
      }
    },
    formatPrice(value) {
      return value.toString().replace(/(\d)(?=(\d{3})+(?!\d))/g, "$1.");
    },
    formatTanggal(value) {
      return this.$moment(value, "YYYY/MM/DD").format("DD MMM YYYY");
    },
    warnacheck: function (status) {
      if (status == "Sended") return "success";
      else if (status == "Pending") return "dangerv";
    },
  },
  mounted() {
    this.readData();
  },
  computed: {
    dateRangeText() {
      return this.datenews.length == 1
        ? this.$moment(this.datenews[0]).format("dddd, MMMM D YYYY")
        : this.momentdate.join(" ~ ");
    },
  },
};
</script>
