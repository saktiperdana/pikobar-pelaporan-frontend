<template>
  <div>
    <v-card
      outlined
    >
      <v-row>
        <v-expansion-panels
          v-model="panelRiwayat"
          multiple
        >
          <v-expansion-panel>
            <v-expansion-panel-header>Update Riwayat Kasus</v-expansion-panel-header>
            <v-expansion-panel-content>
              <ValidationObserver ref="observer">
                <v-form
                  ref="form"
                  lazy-validation
                >
                  <v-row>
                    <v-col
                      cols="12"
                      md="6"
                      sm="12"
                    >
                      <ValidationProvider
                        v-slot="{ errors }"
                        rules="required"
                      >
                        <label class="required">Hasil Pemeriksaan Awal</label>
                        <v-radio-group
                          v-model="formRiwayatPasien.status"
                          :error-messages="errors"
                          row
                        >
                          <v-radio label="ODP" value="ODP" />
                          <v-radio label="PDP" value="PDP" />
                          <v-radio label="POSITIF" value="POSITIF" />
                        </v-radio-group>
                      </ValidationProvider>
                      <ValidationProvider
                        v-slot="{ errors }"
                        rules="required"
                      >
                        <label class="required">Proses Pemeriksaan</label>
                        <v-radio-group
                          v-model="formRiwayatPasien.stage"
                          :error-messages="errors"
                          row
                        >
                          <v-radio label="Proses" value="0" />
                          <v-radio label="Selesai" value="1" />
                        </v-radio-group>
                      </ValidationProvider>
                      <ValidationProvider
                        v-if="formRiwayatPasien.status !== 'ODP'"
                        v-slot="{ errors }"
                      >
                        <label>Hasil Pemeriksaan Akhir</label>
                        <v-radio-group
                          v-model="formRiwayatPasien.final_result"
                          :error-messages="errors"
                          row
                        >
                          <v-radio label="Negatif" value="0" @click.prevent="uncheck('0')" />
                          <v-radio label="Sembuh" value="1" @click.prevent="uncheck('1')" />
                          <v-radio label="Meninggal" value="2" @click.prevent="uncheck('2')" />
                        </v-radio-group>
                      </ValidationProvider>
                      <ValidationProvider
                        v-slot="{ errors }"
                        rules="required"
                      >
                        <label class="required">Lokasi Saat Ini</label>
                        <v-radio-group
                          v-model="formRiwayatPasien.current_location_type"
                          :error-messages="errors"
                          row
                          @change="handleChangeLocationNow"
                        >
                          <v-radio label="Rumah" value="RUMAH" />
                          <v-radio label="Rumah Sakit" value="RS" />
                        </v-radio-group>
                      </ValidationProvider>
                      <div v-if="formRiwayatPasien.current_location_type === 'RUMAH'">
                        <address-region
                          :district-code="formRiwayatPasien.current_location_district_code"
                          :code-district.sync="formRiwayatPasien.current_location_district_code"
                          :sub-district-code="formRiwayatPasien.current_location_subdistrict_code"
                          :code-sub-district.sync="formRiwayatPasien.current_location_subdistrict_code"
                          :village-code="formRiwayatPasien.current_location_village_code"
                          :code-village.sync="formRiwayatPasien.current_location_village_code"
                          :disabled-address="false"
                          :required-address="true"
                        />
                      </div>
                      <ValidationProvider
                        v-if="formRiwayatPasien.current_location_type === 'RUMAH'"
                        v-slot="{ errors }"
                      >
                        <v-label>Alamat Lengkap lokasi saat ini</v-label>
                        <v-text-field
                          v-model="formRiwayatPasien.current_location_address"
                          :error-messages="errors"
                          solo-inverted
                        />
                      </ValidationProvider>
                      <ValidationProvider
                        v-if="formRiwayatPasien.current_location_type === 'RS'"
                        v-slot="{ errors }"
                        rules="required"
                      >
                        <v-autocomplete
                          v-model="formRiwayatPasien.current_location_address"
                          :items="hospitalList"
                          :error-messages="errors"
                          :return-object="true"
                          label="Lokasi Rumah Sakit"
                          menu-props="auto"
                          item-text="name"
                          item-value="name"
                          single-line
                          solo
                          autocomplete
                          @change="onSelectHospital"
                        />
                      </ValidationProvider>
                      <ValidationProvider
                        v-slot="{ errors }"
                      >
                        <v-label>Sumber Pelaporan</v-label>
                        <v-text-field
                          v-model="formRiwayatPasien.report_source"
                          :error-messages="errors"
                          solo-inverted
                        />
                      </ValidationProvider>
                      <ValidationProvider>
                        <v-label>Catatan Tambahan</v-label>
                        <v-textarea
                          v-model="formRiwayatPasien.other_notes"
                          solo
                        />
                      </ValidationProvider>
                    </v-col>
                    <v-col
                      cols="12"
                      md="6"
                      sm="12"
                    >
                      <ValidationProvider>
                        <v-label>Riwayat</v-label>
                        <v-checkbox
                          v-model="formRiwayatPasien.is_went_abroad"
                          label="Dari Luar Negeri"
                        />
                      </ValidationProvider>
                      <ValidationProvider
                        v-if="formRiwayatPasien.is_went_abroad === true"
                        v-slot="{ errors }"
                        rules="required"
                      >
                        <v-text-field
                          v-model="formRiwayatPasien.visited_country"
                          :error-messages="errors"
                          placeholder="Negara Yang Dikunjungi"
                          solo-inverted
                        />
                      </ValidationProvider>
                      <ValidationProvider>
                        <v-checkbox
                          v-model="formRiwayatPasien.is_went_other_city"
                          label="Perjalanan ke luar kota"
                        />
                      </ValidationProvider>
                      <ValidationProvider
                        v-if="formRiwayatPasien.is_went_other_city === true"
                        v-slot="{ errors }"
                        rules="required"
                      >
                        <v-text-field
                          v-model="formRiwayatPasien.visited_city"
                          :error-messages="errors"
                          placeholder="Kota Yang Dikunjungi"
                          solo-inverted
                        />
                      </ValidationProvider>
                      <ValidationProvider>
                        <v-checkbox
                          v-model="formRiwayatPasien.is_contact_with_positive"
                          label="Kontak Dengan Pasien Positif"
                        />
                      </ValidationProvider>
                      <ValidationProvider>
                        <v-text-field
                          v-model="formRiwayatPasien.history_notes"
                          placeholder="Masukkan Riwayat Lainnya Jika Ada"
                          solo-inverted
                        />
                      </ValidationProvider>
                      <label>Tanggal Gejala</label>
                      <select-datetime
                        :datetime="formRiwayatPasien.first_symptom_date"
                        :date-time.sync="formRiwayatPasien.first_symptom_date"
                        :formate-date="formatDate"
                      />
                      <ValidationProvider v-slot="{ errors }">
                        <label>Gejala</label>
                        <div v-for="(item, index) in optionGejala" :key="index">
                          <label class="material-checkbox-custom">
                            <input
                              v-model="formRiwayatPasien.diagnosis"
                              :value="item.value"
                              type="checkbox"
                            >
                            <span v-if="errors.length" class="error--text">{{ item.text }}</span>
                            <span v-else>{{ item.text }}</span>
                          </label>
                        </div>
                        <span
                          v-if="errors.length"
                          class="v-messages error--text"
                        >{{ errors[0] }}</span>
                      </ValidationProvider>
                      <ValidationProvider>
                        <v-text-field
                          v-model="formRiwayatPasien.diagnosis_other"
                          placeholder="Sebutkan gelaja lainnya (jika ada)"
                          solo-inverted
                        />
                      </ValidationProvider>
                    </v-col>
                  </v-row>
                  <v-container fluid>
                    <v-row class="survey-bottom-form">
                      <v-col>
                        <v-btn
                          color="success"
                          bottom
                          style="float: right;"
                          @click="handleSaveHistory"
                        >
                          Update Riwayat
                        </v-btn>
                      </v-col>
                    </v-row>
                  </v-container>
                </v-form>
              </ValidationObserver>
            </v-expansion-panel-content>
          </v-expansion-panel>
        </v-expansion-panels>
      </v-row>
      <v-row>
        <v-expansion-panels
          v-model="panelListRiwayat"
          multiple
        >
          <v-expansion-panel>
            <v-expansion-panel-header>List Riwayat Kasus</v-expansion-panel-header>
            <v-expansion-panel-content>
              <v-simple-table fixed-header height="500px">
                <template v-slot:default>
                  <thead>
                    <tr>
                      <th class="text-left">#</th>
                      <th class="text-left">STATUS</th>
                      <th class="text-left">TAHAPAN</th>
                      <th class="text-left">HASIL</th>
                      <th class="text-left">LOKASI SAAT INI</th>
                      <th class="text-left">TANGGAL DIUPDATE</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="(item, index) in listHistoryCase" :key="item.index">
                      <td>{{ getTableRowNumbering(index) }}</td>
                      <td><status :status="item.status" /></td>
                      <td>
                        <div v-if="item.stage === '0'">
                          Proses
                        </div>
                        <div v-else>
                          Selesai
                        </div>
                      </td>
                      <td>
                        <div v-if=" item.final_result =='0'">
                          Negatif
                        </div>
                        <div v-else-if=" item.final_result =='1'">
                          Sembuh
                        </div>
                        <div v-else-if=" item.final_result =='2'">
                          Meninggal
                        </div>
                        <div v-else>
                          -
                        </div>
                      </td>
                      <td>{{ item.current_location_address }}</td>
                      <td>{{ formatDatetime(item.last_changed, "DD MMMM YYYY") }}</td>
                    </tr>
                  </tbody>
                </template>
              </v-simple-table>
            </v-expansion-panel-content>
          </v-expansion-panel>
        </v-expansion-panels>
      </v-row>
    </v-card>
  </div>
</template>

<script>
import { ValidationObserver, ValidationProvider } from 'vee-validate'
import { optionGejala } from '@/utils/constantVariable'
import { formatDatetime } from '@/utils/parseDatetime'
import { mapGetters } from 'vuex'
export default {
  name: 'EditHistoryCaseForm',
  components: {
    ValidationObserver,
    ValidationProvider
  },
  props: {
    idData: {
      type: String,
      default: null
    }
  },
  data() {
    return {
      formatDate: 'YYYY/MM/DD',
      panelRiwayat: [0],
      panelListRiwayat: [0],
      optionGejala: optionGejala,
      listHistoryCase: null,
      arrayRegion: null
    }
  },
  computed: {
    ...mapGetters('reports', [
      'formRiwayatPasien'
    ]),
    ...mapGetters('region', [
      'hospitalList'
    ])
  },
  async mounted() {
    const response = await this.$store.dispatch('reports/listHistoryCase', this.idData)
    this.listHistoryCase = response.data
  },
  methods: {
    formatDatetime,
    async handleSaveHistory() {
      const valid = await this.$refs.observer.validate()
      if (!valid) {
        return
      }
      await this.$store.dispatch('reports/createHistoryCase', this.formRiwayatPasien)
      await this.$store.dispatch('toast/successToast', 'Data Riwayat Kasus Berhasil Di Perbaharui')
      await this.$store.dispatch('reports/resetRiwayatFormPasien')
      await this.$router.push('/laporan/list')
    },
    getTableRowNumbering(index) {
      return (index + 1)
    },
    onSelectHospital(value) {
      this.formRiwayatPasien.current_hospital_id = value._id
      this.formRiwayatPasien.current_location_address = value.name
    },
    uncheck(value) {
      if (value === this.formRiwayatPasien.final_result) {
        this.formRiwayatPasien.final_result = ''
      } else {
        this.formRiwayatPasien.final_result = value
      }
    },
    handleChangeLocationNow(value) {
      if (value === 'RUMAH') {
        this.formRiwayatPasien.current_location_address = ''
      } else {
        this.formRiwayatPasien.current_hospital_id = ''
        this.formRiwayatPasien.current_location_address = ''
        this.formRiwayatPasien.current_location_district_code = ''
        this.formRiwayatPasien.current_location_subdistrict_code = ''
        this.formRiwayatPasien.current_location_village_code = ''
      }
    }
  }
}
</script>

<style scoped>

</style>
