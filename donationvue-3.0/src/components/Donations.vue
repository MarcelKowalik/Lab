<template>
  <div class="hero">
    <h3 class="vue-title"><i class="fa fa-list" style="padding: 3px"></i>{{messagetitle}}</h3>
    <div id="app1">
      <v-client-table :columns="columns" :data="donations" :options="options">
        <a slot="upvote" slot-scope="props" class="fa fa-thumbs-up fa-2x" @click="upvote(props.row._id)"></a>
        <a slot="edit" slot-scope="props" class="fa fa-edit fa-2x" @click="editDonation(props.row._id)"></a>
        <a slot="remove" slot-scope="props" class="fa fa-trash-o fa-2x" @click="deleteDonation(props.row._id)"></a>
      </v-client-table>
    </div>
  </div>
</template>

<script>
import DonationService from '@/services/DonationService'
import Vue from 'vue'
import VueTables from 'vue-tables-2'

Vue.use(VueTables.ClientTable, {compileTemplates: true, filterByColumn: true})

export default {
  name: 'Donations',
  data () {
    return {
      messagetitle: ' Donations List ',
      donations: [],
      props: ['_id'],
      errors: [],
      columns: ['_id', 'paymenttype', 'amount', 'upvotes', 'upvote', 'edit', 'remove'],
      options: {
        perPage: 10,
        filterable: ['paymenttype', 'amount', 'upvotes'],
        sortable: ['upvotes'],
        headings: {
          _id: 'ID',
          paymenttype: 'Payment Type',
          amount: 'Amount',
          upvotes: 'Upvotes'
        }
      }
    }
  },
  // Fetches Donations when the component is created.
  created () {
    this.loadDonations()
  },
  methods: {
    loadDonations: function () {
      DonationService.fetchDonations()
        .then(response => {
          // JSON responses are automatically parsed.
          this.donations = response.data
          console.log(this.donations)
        })
        .catch(error => {
          this.errors.push(error)
          console.log(error)
        })
    },
    upvote: function (id) {
      DonationService.upvoteDonation(id)
        .then(response => {
          // JSON responses are automatically parsed.
          this.loadDonations()
          console.log(response)
        })
        .catch(error => {
          this.errors.push(error)
          console.log(error)
        })
    },
    editDonation: function (id) {
      this.$router.params = id
      this.$router.push('edit')
    },
    deleteDonation: function (id) {
      this.$swal({
        title: 'Are you totaly sure?',
        text: 'You can\'t Undo this action',
        type: 'warning',
        showCancelButton: true,
        confirmButtonText: 'OK Delete it',
        cancelButtonText: 'Cancel',
        showCloseButton: true
        // showLoaderOnConfirm: true
      }).then((result) => {
        console.log('SWAL Result : ' + result.value)
        if (result.value === true) {
          DonationService.deleteDonation(id)
            .then(response => {
              // JSON responses are automatically parsed.
              this.message = response.data
              console.log(this.message)
              this.loadDonations()
              // Vue.nextTick(() => this.$refs.vuetable.refresh())
              this.$swal('Deleted', 'You successfully deleted this Donation ' + JSON.stringify(response.data, null, 5), 'success')
            })
            .catch(error => {
              this.$swal('ERROR', 'Something went wrong trying to Delete ' + error, 'error')
              this.errors.push(error)
              console.log(error)
            })
        } else {
          console.log('SWAL Else Result : ' + result.value)
          this.$swal('Cancelled', 'Your Donation still Counts!', 'info')
        }
      })
    }
  }
}
</script>

<style scoped>
  #app1 {
    width: 60%;
    margin: 0 auto;
  }
  .vue-title {
    margin-top: 30px;
    text-align: center;
    font-size: 45pt;
    margin-bottom: 10px;
  }
</style>
