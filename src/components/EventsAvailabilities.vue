<template>
  <div>
    <button @click="runTest">Run Test</button>
    <div v-if="fromDate && toDate">
      <h4>Availabilities from {{formatDate(this.fromDate)}} to {{formatDate(this.toDate)}}</h4 >
      
      <p v-html="textAvailabilities"></p>
    </div> 
  </div>
</template>

<script>
import { DateTime } from "luxon";

export default {
  name: 'eventsAvailabilities',
  data() {
    return {
      eventList: [],
      fromDate: null,
      toDate: null,
      textAvailabilities:""
    }
  },
  methods:{
    createEvent: function (opening, recurring, startDate, endDate) {
      let event = {
        opening,recurring,startDate,endDate
      }
      this.eventList.push(event);
    },
    formatDate: (date) => {
      return date ? date.toLocaleString(DateTime.DATETIME_SHORT) : ""
    },
    compareLuxonDates : (a, b) => {
      return a.toMillis() - b.toMillis()
    },
    checkAvailabilities: function() {
      let availabilities = [];
      if(this.fromDate && this.toDate){
      
        const openings = this.eventList.filter((item) => {
          return item.opening;
        })

        const interventions  = this.eventList.filter((item) => {
          return !item.opening;
        })

        //Fill availabilities until toDate 
        openings.forEach((opening) => {
          let openingFromDate = opening.startDate;
          let openingEndDate = opening.endDate;
          let weekDayOfRecurringEvent = openingFromDate.weekday;

          //Fill recuring openings
          if(opening.recurring && openingFromDate <= this.fromDate){
            let date1 = this.fromDate;
            let date2 = this.toDate;
            
            while(date1 <= date2){
              if(weekDayOfRecurringEvent === date1.weekday){
                let openingFromDateRecurring = openingFromDate.set({ day: date1.day })
                let openingEndDateRecurring = openingEndDate.set({ day: date1.day })
                while ( openingFromDateRecurring < openingEndDateRecurring ) {
                  availabilities.push(openingFromDateRecurring)
                  openingFromDateRecurring = openingFromDateRecurring.plus({ minutes: 30 });
                }
              }
              date1 = date1.plus({ days: 1 });
            }  
          }
          //Fill single openings
          else if(!opening.recurring && openingFromDate >= this.fromDate && openingEndDate <= this.toDate){
            while ( openingFromDate < openingEndDate ) {
              availabilities.push(openingFromDate)
              openingFromDate = openingFromDate.plus({ minutes: 30 });
            }
          }
        });

        availabilities.sort(this.compareLuxonDates)

        //Compare availabilities with interventions and remove items from availabilities
        interventions.forEach((intervention) => {
          let interventionFromDate = intervention.startDate;
          let interventionEndDate = intervention.endDate;
          if(interventionFromDate >= this.fromDate && interventionEndDate <= this.toDate){
            while (interventionFromDate < interventionEndDate) {
              availabilities.forEach((availability,index) => {
                if(interventionFromDate.valueOf() === availability.valueOf()){
                  availabilities.splice(index, 1); 
                }
              })
              interventionFromDate = interventionFromDate.plus({ minutes: 30 });
            }
          }    
        });

      }
        
      return availabilities;
    },
    runTest: function() {
      //Fill searching dates
      this.fromDate = DateTime.local(2016,7,4,10,0);
      this.toDate = DateTime.local(2016,7,10,10,0);

      //get the list of availabilities dates
      const availabilities = this.checkAvailabilities();

      let text = "";
      if(availabilities.length) {
          //Construction of the log message
          availabilities.forEach(function(availability,index){
            if(index === 0) {
                text += "I'm available from "+availability.toFormat('MMMM dd')+", at "
            }
            
            if(availabilities[index+1] && availabilities[index+1].day === availability.day && availabilities[index+2] &&  availabilities[index+2].day === availabilities[index+1].day){
                text += availability.toFormat('HH:mm') + ", ";
            }
            else if(availabilities[index+2] && availabilities[index+2].day === availabilities[index+1].day){
                text += availability.toFormat('HH:mm') + " ";
            }
            else if(availabilities[index+2] && availabilities[index+2].day !== availabilities[index+1].day){
                text += availability.toFormat('HH:mm') + " and ";
            }
            else if(!availabilities[index+2] && availabilities[index+1]){
                text += availability.toFormat('HH:mm') + " and ";
            }
            else if(!availabilities[index+1]){
                text += availability.toFormat('HH:mm');
            }

            if(availabilities[index+1] && availabilities[index+1].day !== availability.day)
            {
                text += "<br>I'm also available from "+availabilities[index+1].toFormat('MMMM dd')+", at ";
            }
          });
          text += "<br>I'm not available any other time !";
      }
      else{
          text = "I don't have any availability!"
      }

      this.textAvailabilities = text;
    }
  },
  created (){
    var startDate = DateTime.local(2016,7,1,10,30); // July 1st, 10:30
    var endDate = DateTime.local(2016,7,1,14,0); // July 1st, 14:00

    this.createEvent(true, true, startDate, endDate); // weekly recurring opening in calendar

    startDate = DateTime.local(2016,7,8,11,30); // July 8th 11:30
    endDate = DateTime.local(2016,7,8,12,30); // July 8th 12:30
    this.createEvent(false, false, startDate, endDate); // intervention scheduled

    //ADD MORE EVENTS TO TEST
    // startDate =  DateTime.local(2016,7,5,10,30); // July 5th, 10:30
    // endDate =  DateTime.local(2016,7,5,14,0); // July 5th, 14:00
    // this.createEvent(true, false, startDate, endDate); // single opening in calendar

    // startDate =  DateTime.local(2016,7,9,10,30); // July 9th, 10:30
    // endDate =  DateTime.local(2016,7,9,14,0); // July 9th, 14:00
    // this.createEvent(true, false, startDate, endDate); // single opening in calendar

    // startDate =  DateTime.local(2016,7,5,13,0); // July 5th 11:30
    // endDate =  DateTime.local(2016,7,5,14,0); // July 5th 12:30
    // this.createEvent(false, false, startDate, endDate); // intervention scheduled
  }
}
</script>
