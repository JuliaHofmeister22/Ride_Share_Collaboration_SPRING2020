<template>
  <v-container>
    <div>
      <h4 class="display-1">Current Rides</h4>

       <v-text-field
        v-model="search"
        append-icon="mdi-magnify"
        label="Search for a ride with a specific license number, location, or date"
        single-line
        hide-details
      ></v-text-field>

      <v-data-table
        class="elevation-1"
        v-bind:headers="headers"
        v-bind:items="rides"
        v-bind:search="search"
        loading:
        loading-text="Loading... Please wait"
      >

       <!--put this above if you want the loading bar to show.  How do we turn off loading after data has been loaded to the table?
        loading:
        loading-text="Loading... Please wait"-->
        <template v-slot:item="{ item }">
          <tr v-bind:class="itemClass(item)">
            <td>{{ item.date }}</td>
            <td>{{ item.time }}</td>
            <td>{{ item.distance }}</td>
            <td>{{ item.fuelprice }}</td>
            <td>{{ item.fee }}</td>
            <td>{{ item.vehicleid }}</td>
            <td>{{ item.fromlocation }}</td>
            <td>{{ item.tolocation }}</td>
            <td>{{ item.passengers }}</td>
            <td>{{ item.drivers }}</td>
            <td>
              <v-icon small @click="showDialog(item.id)">mdi-delete</v-icon>
            </td>
          </tr>
        </template>
      </v-data-table>

      <v-dialog v-model="dialog.visible" max-width="400">
        <v-card>
          <v-card-title class="headline">Cancel a Ride</v-card-title>
          <v-card-text>
            Are you sure you want to cancel this ride? <!--ID={{ dialog.rideId }}-->
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="cyan darken-1" class="white--text" @click="cancelRide(dialog.rideId)">
              Yes
            </v-btn>
            <v-btn color="cyan darken-1" class="white--text" @click="hideDialog()">
              No
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>


      <v-snackbar v-model="snackbar.show">
        {{ snackbar.text }}
        <v-btn color="cyan darken-1" text @click="snackbar.show = false">
          Close
        </v-btn>
      </v-snackbar>
    </div>
  </v-container>
</template>

<script>
export default {
  name: "Rides",

  data: function() {
    return {
      search: '',
      headers: [
        { text: "Date", value: "date" },
        { text: "Time", value: "time" },
        { text: "Distance", value: "distance" },
        { text: "Fuel Price", value: "fuelprice" },
        { text: "Fee", value: "fee" },
        { text: "Vehicle ID", value: "vehicleid" },
        { text: "From Location", value: "fromlocation" },
        { text: "To Location", value: "tolocation" },
        { text: "Passengers", value: "passengers" },
        { text: "Drivers", value: "drivers" },
      ],
      rides: [],

      snackbar: {
        show: false,
        text: "",
      },

      dialog: {
        visible: false,
        rideId: NaN,
      },
    };
  },

 mounted: function() {
    //prints out ride information
    this.$axios.get("/rides").then((response) => {
      this.rides = response.data.details.map((ride) => ({
        id: ride.id,
        date: new Date(ride.date).toDateString(),
        time: ride.time,
        distance: `${ride.distance} mi`,
        fuelprice: `$ ${ride.fuelprice}`,
        fee: `$ ${ride.fee}`,
        vehicleid: ride.vehicle.licensenumber,
        fromlocation: `${ride.fromlocation.city}, ${ride.fromlocation.state}`,
        tolocation: `${ride.tolocation.city}, ${ride.tolocation.state}`,
        passengers: ride.passengers.map(p => `${p.firstname} ${p.lastname}`),
        drivers: ride.drivers.map(d => `${d.firstname} ${d.lastname}`),
      }));
      console.log(this.rides);
    });
  },

  
  methods: {
    //show dialog when delete button is pusshed
    showDialog: function (rideId) {
      this.dialog.rideId = rideId;
      this.dialog.visible = true;
    },

    hideDialog: function () {
      this.dialog.visible = false;
    },

    // Display a snackbar message.
    showSnackbar(text) {
      this.snackbar.text = text;
      this.snackbar.show = true;
    },

    // Calculate the CSS class for an item
    itemClass(item) {
      const currentAccount = this.$store.state.currentAccount;
      if (currentAccount && currentAccount.id === item.id) {
        return "currentAccount";
      }
    },

    // Delete a ride.
    cancelRide(rideId) {
      this.$axios
        .delete(
          `/rides/${rideId}`
        )
        .then((response) => {
          try {
            if (response.data.ok) {
              // The delete operation worked on the server; delete the local account
              // by filtering the deleted account from the list of accounts.
              this.rides = this.rides.filter(
                (ride) => ride.id !== rideId
              );
              console.log("YAY it worked");
            }
            console.log(response);
          } catch (e) {
            console.log(e.message);
          }
          this.hideDialog();
        });
    },
  },
};
</script>

<style>
.currentAccount {
  background: lightcoral;
}
</style>
