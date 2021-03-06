<template>
  <div class="q-ma-md">
    <div v-if="$route.params.action" class="q-mx-md q-mt-md text-center caption">
      {{$route.params.action.toUpperCase()}} PREACHER / MINISTER
    </div>
    <div v-if="$route.params.action === 'add'">
      <circuitselect v-if="$route.params.action === 'add'" class="q-ma-md" :perms="['editor','admin']" showme="1"></circuitselect>
      <div class="card q-ma-md bg-lightgrey">
        <q-input v-if="$route.params.action === 'add'" outlined ref="search" @input="searchdb" v-model="search" debounce="500" placeholder="find the preacher or minister's name">
          <template v-slot:append>
            <q-icon name="fa fa-search" />
          </template>
        </q-input>
        <div class="q-ma-md" v-if="individualOptions.length">
          <q-select label="Choose an existing person" v-model="form.individual_id" :options="individualOptions" map-options emit-value/>
        </div>
        <div class="text-center" v-if="search.length > 2">
          <q-btn color="black" @click="modalopen=true" label="Or add a new person"></q-btn>
        </div>
      </div>
    </div>
    <div v-else class="text-center q-my-md caption">
      {{person.title}} {{person.firstname}} {{person.surname}}
    </div>
    <div v-if="form.status !== 'minister'" class="q-ma-md">
      <q-input outlined label="Year on full plan (leave blank if not on full plan)" v-model="form.inducted" />
    </div>
    <div class="q-ma-md">
      <q-select @input="populateTags(form.status)" outlined label="Status" v-model="form.status" :options="[{ label: 'Biblewoman', value: 'biblewoman' }, { label: 'Deacon', value: 'deacon' }, { label: 'Evangelist', value: 'evangelist' }, { label: 'Local preacher', value: 'preacher' }, { label: 'Minister', value: 'minister' }]" map-options emit-value/>
    </div>
    <div class="q-ma-md">
      <q-select multiple outlined use-chips label="Roles" v-model="roles" :options="roleOptions" map-options emit-value/>
    </div>
    <div class="q-ma-md">
      <q-select outlined label="Active" v-model="form.active" :options="[{ label: 'Yes', value: 'yes' }, { label: 'No', value: 'no' }]" map-options emit-value/>
    </div>
    <div class="q-ma-md text-center">
      <q-btn color="primary" @click="submit">OK</q-btn>
      <q-btn class="q-ml-md" color="secondary" @click="$router.back()">Cancel</q-btn>
      <q-btn v-if="$route.params.action === 'edit'" class="q-ml-md" color="black" @click="deletePerson">Delete</q-btn>
    </div>
    <q-dialog minimized full-width v-model="modalopen">
      <q-card class="q-pa-lg">
        <div class="caption text-center">Add a new individual</div>
        <q-input class="q-my-sm" outlined hide-bottom-space error-message="First name is required" label="First name" v-model="person.firstname" :rules="[ val => val.length >= 1 ]"/>
        <q-input class="q-my-sm" outlined hide-bottom-space error-message="Surname is required" label="Surname" v-model="person.firstname" :rules="[ val => val.length >= 1 ]"/>
        <q-input class="q-my-sm" outlined hide-bottom-space error-message="Cellphone must be numeric" label="Cellphone" v-model="person.cellphone" :rules="[ val => val > 1000000 ]"/>
        <q-select class="q-my-sm" outlined label="Sex" v-model="person.sex" :options="[{ label: 'female', value: 'female' }, { label: 'male', value: 'male' }]" map-options emit-value/>
        <q-select class="q-my-sm" outlined label="Title" v-model="person.title" :options="[{ label: 'Dr', value: 'Dr' }, { label: 'Mr', value: 'Mr' }, { label: 'Mrs', value: 'Mrs' }, { label: 'Ms', value: 'Ms' }, { label: 'Prof', value: 'Prof' }, { label: 'Rev', value: 'Rev' }]" map-options emit-value/>
        <q-select class="q-my-sm" outlined label="Society" v-model="person.society_id" :options="societyOptions" @blur="$v.person.society_id.$touch()" :error="$v.person.society_id.$error" map-options emit-value/>
        <div class="text-center">
          <q-btn class="q-mt-md" color="primary" @click="addperson()" label="Save" />
          <q-btn class="q-mt-md q-ml-md" color="black" @click="modalopen = false" label="Cancel" />
        </div>
      </q-card>
    </q-dialog>
  </div>
</template>

<script>
import { between, required, numeric } from 'vuelidate/lib/validators'
import circuitselect from './../Circuitselect'
export default {
  data () {
    return {
      search: '',
      circuits: [],
      showdropdown: false,
      individualOptions: [],
      societyOptions: [],
      roleOptions: [],
      modalopen: false,
      form: {
        status: '',
        inducted: '',
        individual_id: '',
        circuit_id: '',
        roles: [],
        active: 'yes'
      },
      roles: [],
      person: {
        firstname: '',
        surname: '',
        title: 'Ms',
        society_id: '',
        cellphone: '',
        sex: 'female'
      }
    }
  },
  validations: {
    form: {
      inducted: { between: between(1900, 2050) },
      individual_id: { required },
      roles: { required }
    },
    person: {
      firstname: { required },
      surname: { required },
      cellphone: { numeric },
      society_id: { required }
    }
  },
  components: {
    'circuitselect': circuitselect
  },
  methods: {
    submit () {
      this.$q.loading.show()
      if (this.$route.params.action === 'add') {
        this.$axios.defaults.headers.common['Authorization'] = 'Bearer ' + this.$store.state.token
        this.$axios.post(process.env.API + '/circuits/' + this.$store.state.select + '/people',
          {
            personable_type: 'Bishopm\\Churchnet\\Models\\Circuit',
            personable_id: this.$store.state.select,
            inducted: this.form.inducted,
            active: this.form.active,
            individual_id: this.form.individual_id.value,
            roles: this.roles,
            status: this.form.status
          })
          .then(response => {
            this.$q.loading.hide()
            this.$q.notify('Preacher added')
            this.$router.push({ name: 'preachers' })
          })
          .catch(function (error) {
            console.log(error)
            this.$q.loading.hide()
          })
      } else {
        this.$axios.defaults.headers.common['Authorization'] = 'Bearer ' + this.$store.state.token
        this.$axios.post(process.env.API + '/circuits/' + this.form.circuit_id + '/people/' + this.form.id,
          {
            personable_type: 'Bishopm\\Churchnet\\Models\\Circuit',
            personable_id: this.$store.state.select,
            inducted: this.form.inducted,
            individual_id: this.form.individual_id.value,
            roles: this.roles,
            active: this.form.active,
            status: this.form.status
          })
          .then(response => {
            this.$q.loading.hide()
            this.$q.notify('Preacher updated')
            this.$router.push({ name: 'preachers' })
          })
          .catch(function (error) {
            console.log(error)
            this.$q.loading.hide()
          })
      }
    },
    deletePerson () {
      this.$axios.defaults.headers.common['Authorization'] = 'Bearer ' + this.$store.state.token
      this.$axios.delete(process.env.API + '/people/' + this.form.id)
        .then(response => {
          this.$q.notify('Person deleted')
          this.$router.push({ name: 'preachers' })
        })
        .catch(function (error) {
          console.log(error)
        })
    },
    populateTags (category) {
      this.roleOptions = []
      for (var rkey in this.form.alltags[category]) {
        var newitem = {
          label: this.form.alltags[category][rkey].name,
          value: this.form.alltags[category][rkey].id
        }
        this.roleOptions.push(newitem)
      }
      this.roles = []
      for (var tkey in this.form.tags) {
        this.roles.push(this.form.tags[tkey].tag_id)
      }
    },
    searchdb () {
      if (this.search.length > 2) {
        this.$q.loading.show()
        this.$axios.defaults.headers.common['Authorization'] = 'Bearer ' + this.$store.state.token
        this.$axios.post(process.env.API + '/individuals/searchnp',
          {
            search: this.search,
            circuit: this.$store.state.select
          })
          .then(response => {
            this.individualOptions = []
            for (var ikey in response.data) {
              if (response.data[ikey.title]) {
                var newitem1 = {
                  label: response.data[ikey].surname + ', ' + response.data[ikey].title + ' ' + response.data[ikey].firstname + ' (' + response.data[ikey].household.society.society + ')',
                  value: response.data[ikey].id
                }
                this.individualOptions.push(newitem1)
              } else {
                var newitem2 = {
                  label: response.data[ikey].surname + ', ' + response.data[ikey].firstname + ' (' + response.data[ikey].household.society.society + ')',
                  value: response.data[ikey].id
                }
                this.individualOptions.push(newitem2)
              }
            }
            if (this.individualOptions.length) {
              this.form.individual_id = this.individualOptions[0].value
            }
            this.$q.loading.hide()
          })
          .catch(function (error) {
            console.log(error)
            this.$q.loading.hide()
          })
      } else {
        this.individualOptions = []
        this.form.individual_id = ''
      }
    },
    addperson () {
      this.$v.person.$touch()
      if (this.$v.person.$error) {
        this.$q.notify('Please check for errors!')
      } else {
        this.$axios.defaults.headers.common['Authorization'] = 'Bearer ' + this.$store.state.token
        this.$axios.post(process.env.API + '/combined',
          {
            firstname: this.person.firstname,
            surname: this.person.surname,
            phone: this.person.cellphone,
            society_id: this.person.society_id,
            sex: this.person.sex,
            title: this.person.title,
            adduser: 'no'
          })
          .then(response => {
            this.search = ''
            this.modalopen = false
            this.$q.notify('New individual has been added')
            var newp = {
              label: response.data.surname + ', ' + response.data.firstname + ' (' + response.data.society + ')',
              value: response.data.id
            }
            this.individualOptions.push(newp)
            this.form.individual_id = response.data.id
          })
          .catch(function (error) {
            console.log(error)
          })
      }
    }
  },
  mounted () {
    this.circuits.push(this.$store.state.select)
    if (this.$route.params.action === 'edit') {
      this.$axios.defaults.headers.common['Authorization'] = 'Bearer ' + this.$store.state.token
      this.$axios.get(process.env.API + '/tags')
        .then((response) => {
          this.person = JSON.parse(this.$route.params.preacher)
          this.form = JSON.parse(this.$route.params.preacher).person
          this.form.alltags = response.data
          this.populateTags(this.form.status)
        })
        .catch(function (error) {
          console.log(error)
        })
    } else {
      this.$axios.defaults.headers.common['Authorization'] = 'Bearer ' + this.$store.state.token
      this.$axios.post(process.env.API + '/societies/search',
        {
          search: '',
          circuits: this.circuits
        })
        .then(response => {
          for (var sndx in response.data) {
            var newitem = {
              label: response.data[sndx].society,
              value: response.data[sndx].id
            }
            this.societyOptions.push(newitem)
          }
          this.$axios.defaults.headers.common['Authorization'] = 'Bearer ' + this.$store.state.token
          this.$axios.get(process.env.API + '/tags')
            .then((response) => {
              this.form.alltags = response.data
              this.populateTags(this.form.status)
            })
            .catch(function (error) {
              console.log(error)
            })
        })
        .catch(function (error) {
          console.log(error)
        })
    }
  }
}
</script>

<style>
  a {
    text-decoration: none;
    color:white;
  }
  .bg-lightgrey {
    background-color: #eee;
    padding-top:10px;
    padding-bottom:10px;
  }
</style>
