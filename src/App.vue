<template>
  <v-app>
    <v-app-bar app clipped-left color="primary">
      <v-avatar width="60">
        <img src="./assets/logo.png" alt="Logo" />
      </v-avatar>
      <v-toolbar-title class="white--text">Kur me dal?</v-toolbar-title>
    </v-app-bar>
    <v-content>
      <v-container fluid>
        <v-row
          class="fill-height d-sm-none d-none d-md-flex"
          align="center"
          justify="center"
        >
          <v-col cols="10">
            <v-card class="pa-4 fill-height">
              <v-sheet height="64">
                <v-toolbar flat>
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
                  <v-select
                    @change="setEvents"
                    :items="numbers"
                    dense
                    v-model="selectedNumber"
                    class="pt-2 pr-2"
                    label="Zgjedh numrin e parafundit të letërnjoftimit"
                  ></v-select>
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
              <v-sheet height="50rem">
                <v-calendar
                  ref="calendar"
                  v-if="events.length"
                  v-model="focus"
                  :weekdays="weekdays"
                  color="primary"
                  :event-height="40"
                  :interval-height="30"
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
                >
                  <template v-slot:event="{ event }">
                    <v-row align="center" justify="center">{{
                      event.name
                    }}</v-row>
                  </template>
                </v-calendar>
                <v-menu
                  v-model="selectedOpen"
                  :close-on-content-click="false"
                  :activator="selectedElement"
                  offset-x
                >
                </v-menu>
              </v-sheet>
            </v-card>
          </v-col>
        </v-row>
        <v-row
          align="center"
          class="d-flex d-sm-flex d-md-none"
          justify="center"
        >
          <v-select
            @change="setEvents"
            :items="numbers"
            dense
            v-model="selectedNumber"
            class="pa-4"
            label="Zgjedh numrin e parafundit të letërnjoftimit"
          ></v-select>
        </v-row>
        <v-row class="d-flex d-sm-flex d-md-none">
          <v-card min-width="100vw" class="mx-auto">
            <v-list subheader>
              <v-subheader>Prill</v-subheader>

              <template v-for="(item, index) in listEvents.april">
                <v-list-item :key="item.date + item.name" two-line>
                  <v-list-item-avatar
                    :color="item.selected ? 'pink' : 'blue lighten-5'"
                  >
                    <v-row align="center" justify="center">
                      <div :class="{ 'white--text': item.selected }">
                        {{ item.date }}
                      </div>
                    </v-row>
                  </v-list-item-avatar>

                  <v-list-item-content>
                    <v-list-item-title v-text="item.day"></v-list-item-title>
                    <v-list-item-subtitle
                      v-text="item.name"
                    ></v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
                <v-divider
                  v-if="index + 1 < listEvents.april.length"
                  :key="index"
                ></v-divider>
              </template>
            </v-list>
            <v-list subheader>
              <v-subheader>Maj</v-subheader>

              <template v-for="(item, index) in listEvents.may">
                <v-list-item :key="item.date + item.name" two-line>
                  <v-list-item-avatar
                    :color="item.selected ? 'pink' : 'blue lighten-5'"
                  >
                    <v-row align="center" justify="center">
                      <div :class="{ 'white--text': item.selected }">
                        {{ item.date }}
                      </div>
                    </v-row>
                  </v-list-item-avatar>

                  <v-list-item-content>
                    <v-list-item-title v-text="item.day"></v-list-item-title>
                    <v-list-item-subtitle
                      v-text="item.name"
                    ></v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
                <v-divider
                  v-if="index + 1 < listEvents.april.length"
                  :key="index"
                ></v-divider>
              </template>
            </v-list>
          </v-card>
        </v-row>
      </v-container>
    </v-content>
    <v-footer app>
      <v-spacer></v-spacer>
      <a href="./assets/05-1898.pdf" target="_blank">Vendimi</a>
    </v-footer>
  </v-app>
</template>

<script>
import events from "./orari_i_levizjes.json";
export default {
  data: () => ({
    focus: "",
    type: "month",
    typeToLabel: {
      month: "Muaj",
      week: "Javë",
      day: "Dite"
    },
    start: null,
    end: null,
    selectedNumber: localStorage.getItem("selectedNumber") || "0",
    weekdays: [1, 2, 3, 4, 5, 6, 0],
    daysMap: ["D", "Ha", "Ma", "Mr", "Ej", "Pr", "Sh"],
    daysMapL: [
      "E Diel",
      "E Hënë",
      "E Martë",
      "E Mërkurë",
      "E Enjte",
      "E Premte",
      "E Shtunë"
    ],
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
    listEvents: {
      april: [],
      may: []
    },
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
  created() {
    this.setEvents();
  },
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
      localStorage.setItem("selectedNumber", this.selectedNumber);
      const ev = [];
      const listEv = {
        april: [],
        may: []
      };
      Object.keys(events).forEach(dt => {
        const parts = dt.split("-");
        const duration = events[dt][this.selectedNumber];
        const mnth = parts[1] === "04" ? "april" : "may";
        listEv[mnth].push({
          date: parts[0],
          name: `${duration.from} - ${duration.to}`,
          day: this.daysMapL[
            new Date(`${parts[2]}-${parts[1]}-${parts[0]}`).getDay()
          ],
          selected:
            new Date().toDateString() ===
            new Date(`${parts[2]}-${parts[1]}-${parts[0]}`).toDateString()
        });
        const date = `${parts[2]}-${parts[1]}-${parts[0]}`;
        const min = new Date(`${date}T${duration.from}:00`);
        const max = new Date(`${date}T${duration.to}:59`);
        ev.push({
          name: `${duration.from} - ${duration.to}`,
          start: this.formatDate(min, true),
          end: this.formatDate(max, true),
          color: "pink"
        });
      });
      this.events = ev;
      this.listEvents = listEv;
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
