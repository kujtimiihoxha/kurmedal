<template>
  <v-app>
    <v-app-bar app clipped-left color="primary">
      <v-avatar width="60">
        <img src="./assets/logo.png" alt="Logo" />
      </v-avatar>
      <v-toolbar-title class="white--text">Kur me dal?</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-menu bottom right>
        <template v-slot:activator="{ on }">
          <v-btn outlined color="white" v-on="on">
            <span>{{ langToLabel[lang] }}</span>
            <v-icon right>mdi-menu-down</v-icon>
          </v-btn>
        </template>
        <v-list>
          <v-list-item @click="setLanguage('sq')">
            <v-list-item-title>Shqip</v-list-item-title>
          </v-list-item>
          <v-list-item @click="setLanguage('en')">
            <v-list-item-title>English</v-list-item-title>
          </v-list-item>
          <v-list-item @click="setLanguage('srb')">
            <v-list-item-title>Srpski</v-list-item-title>
          </v-list-item>
        </v-list>
      </v-menu>
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
                    {{ todayMap[lang] }}
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
                    :label="inputMap[lang]"
                  ></v-select>
                  <v-menu bottom right>
                    <template v-slot:activator="{ on }">
                      <v-btn outlined color="grey darken-2" v-on="on">
                        <span>{{ typeToLabel[lang][type] }}</span>
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
                  :weekday-format="d => daysMap[lang][d.weekday]"
                  :month-format="m => monthMap[lang][m.month - 1]"
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
            :label="inputMap[lang]"
          ></v-select>
        </v-row>
        <v-row class="d-flex d-sm-flex d-md-none">
          <v-card min-width="100vw" class="mx-auto">
            <v-list subheader>
              <v-subheader>{{ monthMap[lang][3] }}</v-subheader>

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
                    <v-list-item-title
                      v-text="daysMapL[lang][item.day]"
                    ></v-list-item-title>
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
              <v-subheader>{{ monthMap[lang][4] }}</v-subheader>

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
                    <v-list-item-title
                      v-text="daysMapL[lang][item.day]"
                    ></v-list-item-title>
                    <v-list-item-subtitle
                      v-text="item.name"
                    ></v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
                <v-divider
                  v-if="index + 1 < listEvents.may.length"
                  :key="index"
                ></v-divider>
              </template>
            </v-list>
          </v-card>
        </v-row>
      </v-container>
    </v-content>
    <v-footer app>
      <a
        href="https://github.com/kujtimiihoxha/kurmedal"
        target="_blank"
        class="pt-2"
      >
        <svg id="i-github" viewBox="0 0 64 64" width="25" height="25">
          <path
            stroke-width="0"
            fill="black"
            d="M32 0 C14 0 0 14 0 32 0 53 19 62 22 62 24 62 24 61 24 60 L24 55 C17 57 14 53 13 50 13 50 13 49 11 47 10 46 6 44 10 44 13 44 15 48 15 48 18 52 22 51 24 50 24 48 26 46 26 46 18 45 12 42 12 31 12 27 13 24 15 22 15 22 13 18 15 13 15 13 20 13 24 17 27 15 37 15 40 17 44 13 49 13 49 13 51 20 49 22 49 22 51 24 52 27 52 31 52 42 45 45 38 46 39 47 40 49 40 52 L40 60 C40 61 40 62 42 62 45 62 64 53 64 32 64 14 50 0 32 0 Z"
          />
        </svg>
      </a>
      <v-spacer></v-spacer>
      <a href="./assets/05-1898.pdf" target="_blank">{{ decisionMap[lang] }}</a>
    </v-footer>
  </v-app>
</template>

<script>
import events from "./orari_i_levizjes.json";
export default {
  data: () => ({
    focus: "",
    type: "month",
    todayMap: {
      sq: "Sot",
      en: "Today",
      srb: "Danas"
    },
    decisionMap: {
      sq: "Vendimi",
      en: "Decision",
      srb: "Odluka"
    },
    inputMap: {
      sq: "Zgjedh shifren e parafundit të Nr. Personal",
      srb: "Izaberite vaš predposledni broj vašeg ličnog broja",
      en: "Select the second last number of your Personal ID/ Passport No."
    },
    typeToLabel: {
      sq: {
        month: "Muaj",
        week: "Javë",
        day: "Dite"
      },
      en: {
        month: "Month",
        week: "Week",
        day: "Day"
      },
      srb: {
        month: "Mesec",
        week: "Nedelja",
        day: "Dan"
      }
    },
    lang: localStorage.getItem("language") || "sq",
    langToLabel: {
      sq: "Shqip",
      en: "English",
      srb: "Srpski"
    },
    start: null,
    end: null,
    selectedNumber: localStorage.getItem("selectedNumber") || "0",
    weekdays: [1, 2, 3, 4, 5, 6, 0],
    daysMap: {
      sq: ["D", "Ha", "Ma", "Mr", "Ej", "Pr", "Sh"],
      en: ["SUN", "MON", "TUE", "WED", "THUR", "FRI", "SAT"],
      srb: ["N", "P", "U", "S", "Č", "P", "S"]
    },
    daysMapL: {
      sq: [
        "E Diel",
        "E Hënë",
        "E Martë",
        "E Mërkurë",
        "E Enjte",
        "E Premte",
        "E Shtunë"
      ],
      en: [
        "Sunday",
        "Monday",
        "Tuesday",
        "Wednesday",
        "Thursday",
        "Friday",
        "Saturday"
      ],
      srb: [
        "Nedelja",
        "Ponedeljak",
        "Utorak",
        "Sreda",
        "Četvrtak",
        "Petak",
        "Subota"
      ]
    },
    monthMap: {
      sq: [
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
      en: [
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December"
      ],
      srb: [
        "Januar",
        "Februar",
        "Mart",
        "April",
        "Maj",
        "Jun",
        "Jul",
        "Avgust",
        "Septembar",
        "Oktobar",
        "Novembar",
        "Decembar"
      ]
    },
    selectedEvent: {},
    selectedElement: null,
    selectedOpen: false,
    events: [],
    listEvents: {
      april: [],
      may: []
    },
    numbers: ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9"]
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

      const startMonth = this.monthMap[this.lang][
        this.monthFormatter(start) - 1
      ];
      const endMonth = this.monthMap[this.lang][this.monthFormatter(end) - 1];
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
    setLanguage(l) {
      this.lang = l;
      localStorage.setItem("language", l);
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
          day: new Date(`${parts[2]}-${parts[1]}-${parts[0]}`).getDay(),
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
      this.start = start;
      this.end = end;
    },
    nth(d) {
      return d > 3 && d < 21
        ? "th"
        : ["th", "st", "nd", "rd", "th", "th", "th", "th", "th", "th"][d % 10];
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
