<template>
    <div>
      <!-- header-->
      <div class="header">
        <h1>List of patients</h1>
        <div>
          <nuxt-link to="/patients/add-patient">
            <span class="btn btn-add">+ Add new</span>
          </nuxt-link>
        </div>
      </div>

      <!-- loading-->
      <div v-if="loading">
        <loading-spinner></loading-spinner>
      </div>

      <!-- table  -->
      <table v-else>
        <thead>
          <tr>
            <th>Last name</th>
            <th>First name</th>
            <th>Absolved vaccine</th>
            <th>actions</th>
          </tr>
        </thead>
        <tbody v-if="patients">
          <tr v-for="patient in sortedArray" :key="patient.id">
            <td>{{ patient.last_name }} </td>
            <td>{{ patient.first_name }} </td>
            <td>{{ patient.vaccination_number }}</td>
            <td>
              <nuxt-link :to="'patients/' + patient.id">
                <span class="btn btn-edit">Edit</span>
              </nuxt-link>
              <span class="btn btn-delete" v-on:click="deletePatient(patient.id)">X delete</span>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
</template>

<script>
import config from './ApiConfig'
import axios from 'axios'
import LoadingSpinner from './Loading-spinner.vue'

export default {
  components: { LoadingSpinner },
  name: 'PatientTable',

  data() {
    return {
      config,
      patients: '',
      loading: false
    }
  },

  async created () {
    this.loading = true
    try {
      const res = await axios.get('https://hyperia-hospital.herokuapp.com/patients/', this.config )
      this.patients = res.data
      this.loading = false
    } catch (error) {
      console.log('error bol: ', error)
      this.loading = false
    }
  },

  methods: {
    deletePatient(id) {
      var agree = confirm("Want to delete?");
      if (agree) {
        // Delete the item
        axios.delete(`https://hyperia-hospital.herokuapp.com/${id}`, config)
          .then(response => {
              console.log('malo by uspesne odstranené: ', response)
              // delete item from array
              var index = this.patients.findIndex(function(o){
                  return o.id === id;
              })
              if (index !== -1) this.patients.splice(index, 1);

              this.submiting = false
              // smooth scroll to top
              window.scrollTo({top: 0, behavior: 'smooth'});
          })
          .catch(error => {
              // this.errorMessage = error.message;
              console.error("There was an error!", error);
          });
      }
    }
  },

  computed: {
    sortedArray(){
      return this.patients.sort(function(a, b){
        if(a.last_name < b.last_name) { return -1; }
        if(a.last_name > b.last_name) { return 1; }
        return 0;
      })
    }
  }
}
</script>

<style scoped>
.header{
  display: flex;
  justify-content: space-between;
  align-items: center;
}
table {
  font-family: arial, sans-serif;
  border-collapse: collapse;
  width: 100%;
}
td, th {
  border: 1px solid #dddddd;
  text-align: left;
  padding: 15px;
}

tr:nth-child(even) {
  background-color: #f1f1f1;
}
.btn-edit{
  border: 1px solid rgba(63, 63, 191, 0.4);
  box-shadow: rgba(0, 0, 0, 0.15) 1.95px 1.95px 2.6px;
}
.btn-add{
  border: 1px solid rgba(18, 43, 201, 0.4);
  box-shadow: rgba(99, 99, 99, 0.2) 0px 2px 8px 0px;
}
.btn-delete{
  color: red;
  margin-left: 5px;
  font-size: 11px;
  margin-top: 4px;
  text-decoration: underline;
}
</style>