<template>
  <v-app>
    <v-app-bar app clipped-left>
      <v-toolbar-title>Kur me dal?</v-toolbar-title>
    </v-app-bar>
    <v-content>
      <v-row justify="center">
        <v-col cols="12" md="8" class="pr-2 pl-2 mt-4">
          <v-autocomplete
                  @change="setEvents"
                  :items="numbers"
                  v-model="selectedNumber"
                  label="Zgjedh numrin e parafundit të letërnjoftimit"
                  outlined
          ></v-autocomplete>
        </v-col>
      </v-row>
      <v-row class="fill-height" align="center" justify="center">
        <v-col cols="12" md="10" class="pa-4" style="border: solid lightgray 1px; border-radius: 6px">
          <v-sheet height="64">
            <v-toolbar flat color="white">
              <v-btn
                outlined
                class="mr-4"
                color="grey darken-2"
                @click="setToday"
              >
                Sot
              </v-btn>
              <v-btn fab text small color="grey darken-2" @click="prev">
                <v-icon small>mdi-chevron-left</v-icon>
              </v-btn>
              <v-btn fab text small color="grey darken-2" @click="next">
                <v-icon small>mdi-chevron-right</v-icon>
              </v-btn>
              <v-toolbar-title>{{ title }}</v-toolbar-title>
              <v-spacer></v-spacer>
              <v-menu bottom right>
                <template v-slot:activator="{ on }">
                  <v-btn outlined color="grey darken-2" v-on="on">
                    <span>{{ typeToLabel[type] }}</span>
                    <v-icon right>mdi-menu-down</v-icon>
                  </v-btn>
                </template>
                <v-list>
                  <v-list-item @click="type = 'day'">
                    <v-list-item-title>Ditë</v-list-item-title>
                  </v-list-item>
                  <v-list-item @click="type = 'week'">
                    <v-list-item-title>Jave</v-list-item-title>
                  </v-list-item>
                  <v-list-item @click="type = 'month'">
                    <v-list-item-title>Muaj</v-list-item-title>
                  </v-list-item>
                </v-list>
              </v-menu>
            </v-toolbar>
          </v-sheet>
          <v-sheet class="fill-height">
            <v-calendar
              ref="calendar"
              v-model="focus"
              :weekday="weekday"
              color="primary"
              :events="events"
              :weekday-format="d => daysMap[d.weekday]"
              :month-format="m => monthMap[m.month - 1]"
              :event-color="getEventColor"
              :interval-format="i => i.time"
              :now="today"
              :type="type"
              @click:more="viewDay"
              @click:date="viewDay"
              @change="updateRange"
            ></v-calendar>
            <v-menu
              v-model="selectedOpen"
              :close-on-content-click="false"
              :activator="selectedElement"
              offset-x
            >
            </v-menu>
          </v-sheet>
        </v-col>
      </v-row>
    </v-content>
  </v-app>
</template>

<script>
// export default {
//   name: "App",
//
//   data: () => ({
//     value: null,
//     weekday: [1, 2, 3, 4, 5, 6, 0],
//     daysMap: ["Ha", "Ma", "Mr", "Ej", "Pr", "Sh", "D"]
//   }),
//
//   methods: {
//     daysFormat(day) {
//       return this.daysMap[day.weekday];
//     }
//   },
//   computed: {
//     today() {
//       const d = new Date();
//       const dtf = new Intl.DateTimeFormat("en", {
//         year: "numeric",
//         month: "numeric",
//         day: "2-digit"
//       });
//       const [
//         { value: mo },
//         ,
//         { value: da },
//         ,
//         { value: ye }
//       ] = dtf.formatToParts(d);
//       return `${ye}-${mo}-${da}`;
//     }
//   }
// };
import events from "./orari_i_levizjes.json";
export default {
  data: () => ({
    focus: "",
    type: "week",
    typeToLabel: {
      month: "Muaj",
      week: "Javë",
      day: "Dite"
    },
    start: null,
    end: null,
    selectedNumber: "0",
    weekday: [1, 2, 3, 4, 5, 6, 0],
    daysMap: ["Ha", "Ma", "Mr", "Ej", "Pr", "Sh", "D"],
    monthMap: [
      "Janar",
      "Shkurt",
      "Mars",
      "Prill",
      "Maj",
      "Qershor",
      "Korrik",
      "Gushtë",
      "Shtator",
      "Tetor",
      "Nëntor",
      "Dhjetor"
    ],
    selectedEvent: {},
    selectedElement: null,
    selectedOpen: false,
    events: [],
    numbers: ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9"],
    names: [
      "Meeting",
      "Holiday",
      "PTO",
      "Travel",
      "Event",
      "Birthday",
      "Conference",
      "Party"
    ]
  }),
  computed: {
    title() {
      const { start, end } = this;
      if (!start || !end) {
        return "";
      }

      const startMonth = this.monthMap[this.monthFormatter(start) - 1];
      const endMonth = this.monthMap[this.monthFormatter(end) - 1];
      const suffixMonth = startMonth === endMonth ? "" : endMonth;

      const startYear = start.year;
      const endYear = end.year;
      const suffixYear = startYear === endYear ? "" : endYear;

      const startDay = start.day + this.nth(start.day);
      const endDay = end.day + this.nth(end.day);

      switch (this.type) {
        case "month":
          return `${startMonth} ${startYear}`;
        case "week":
          return `${startMonth} ${startDay} ${startYear} - ${suffixMonth} ${endDay} ${suffixYear}`;
        case "day":
          return `${startMonth} ${startDay} ${startYear}`;
      }
      return "";
    },
    monthFormatter() {
      return this.$refs.calendar.getFormatter({
        timeZone: "UTC",
        month: "numeric"
      });
    },
    today() {
      const d = new Date();
      const dtf = new Intl.DateTimeFormat("en", {
        year: "numeric",
        month: "numeric",
        day: "2-digit"
      });
      const [
        { value: mo },
        ,
        { value: da },
        ,
        { value: ye }
      ] = dtf.formatToParts(d);
      return `${ye}-${mo}-${da}`;
    }
  },
  mounted() {
    this.$refs.calendar.checkChange();
  },
  methods: {
    viewDay({ date }) {
      this.focus = date;
      this.type = "day";
    },
    getEventColor(event) {
      return event.color;
    },
    setToday() {
      this.focus = this.today;
    },
    prev() {
      this.$refs.calendar.prev();
    },
    next() {
      this.$refs.calendar.next();
    },
    setEvents() {
      const ev = [];
      Object.keys(events).forEach(dt => {
        const parts = dt.split("-");
        const date = `${parts[2]}-${parts[1]}-${parts[0]}`;
        const duration = events[dt][this.selectedNumber];
        const min = new Date(`${date}T${duration.from}:00`);
        const max = new Date(`${date}T${duration.to}:59`);
        ev.push({
          name: `Nga ora ${duration.from} deri ${duration.to}`,
          start: this.formatDate(min, true),
          end: this.formatDate(max, true),
          color: "indigo"
        });
      });
      this.events = ev;
    },
    updateRange({ start, end }) {
      this.setEvents();
      // const events = [];
      //
      // const min = new Date(`${start.date}T00:00:00`);
      // const max = new Date(`${end.date}T23:59:59`);
      // const days = (max.getTime() - min.getTime()) / 86400000;
      // const eventCount = this.rnd(days, days + 20);
      //
      // for (let i = 0; i < eventCount; i++) {
      //   const allDay = this.rnd(0, 3) === 0;
      //   const firstTimestamp = this.rnd(min.getTime(), max.getTime());
      //   const first = new Date(firstTimestamp - (firstTimestamp % 900000));
      //   const secondTimestamp = this.rnd(2, allDay ? 288 : 8) * 900000;
      //   const second = new Date(first.getTime() + secondTimestamp);
      //
      //   events.push({
      //     name: this.names[this.rnd(0, this.names.length - 1)],
      //     start: this.formatDate(first, !allDay),
      //     end: this.formatDate(second, !allDay),
      //     color: this.colors[this.rnd(0, this.colors.length - 1)]
      //   });
      // }

      this.start = start;
      this.end = end;
    },
    nth(d) {
      return d > 3 && d < 21
        ? "th"
        : ["th", "st", "nd", "rd", "th", "th", "th", "th", "th", "th"][d % 10];
    },
    rnd(a, b) {
      return Math.floor((b - a + 1) * Math.random()) + a;
    },
    formatDate(a, withTime) {
      return withTime
        ? `${a.getFullYear()}-${a.getMonth() +
            1}-${a.getDate()} ${a.getHours()}:${a.getMinutes()}`
        : `${a.getFullYear()}-${a.getMonth() + 1}-${a.getDate()}`;
    }
  }
};
</script>
