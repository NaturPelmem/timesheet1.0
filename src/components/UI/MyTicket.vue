<template>
      <div v-for="(tickets, numberlesson_name) in groupedTickets" :key="numberlesson_name" :class="isPractice(tickets, numberlesson_name) ? 'ticket practice' : 'ticket'">
    <v-row v-for="ticket in tickets" :key="ticket.id" class="ticket-row">
      <v-col class="ticket-number">
        <div class="ticket-time">{{ ticket.number.starttimelesson.substr(0, 5) }}
          {{ ticket.number.endtimelesson.substr(0, 5) }}
        </div>
        <div class="ticket-number-num">
          <div class="number-num">{{ ticket.number.numberlesson_name }}</div>
          <div class="number-short">{{ ticket.number.short }}</div>
        </div>
      </v-col>
      <v-col class="ticket-theme">{{ ticket.theme ? ticket.theme.theme_name : '' }}</v-col>
      <v-col cols="4" class="ticket-teacher">{{ ticket.teacher ? ticket.teacher.last_name : '' }}
        {{ ticket.teacher ? ticket.teacher.first_name : '' }}
        {{ ticket.teacher ? ticket.teacher.middle_name : '' }}
      </v-col>
      <v-col cols="2" class="ticket-group">{{ ticket.group.group_name }}</v-col>
      <v-col cols="1" class="ticket-subgroup">{{ ticket.subgroup ? ticket.subgroup.subgroups_name : '' }}</v-col>
      <v-col cols="1" class="ticket-cabinet">{{ ticket.cabinet ? ticket.cabinet.cabinet_name : '' }}</v-col>

      <v-col class="ticket-practice_name">{{ ticket.practice_name ? ticket.practice_name.practice_name : '' }}</v-col>
      <v-col cols="2" class="ticket-startpractice">{{ ticket ? ticket.startpractice : '' }}</v-col>
      <v-col cols="2" class="ticket-endpractice">{{ tickets ? ticket.endpractice : '' }}</v-col>
    </v-row>
  </div>
  <v-row class="lazy" v-show="filteredLesson.length === 0">{{ lazy }}</v-row>
</template>

<script>
import axios from "axios";

export default {
  name: "MyTicket",
  components: {},
  data: () => ({
    lesson: [],
    'api': 'https://jaronimo.pythonanywhere.com/api/lessonlist/',
    lazy: 'Пар нет, ленимся дальше 😴',
  }),
  methods: {
    async getLessons() {
      axios.get(this.api).then(
          response => {
            this.lesson = response.data;
          }
      ).catch(error => {
        console.log(error)
      })
    },
    checkVisibility(ticket) {
      const searchQuery = this.$store.state.searchQuery;
      const transfers = this.$store.state.transfers;
      const teacherFullName = (ticket.teacher?.last_name || '') + ' ' + (ticket.teacher?.first_name || '') + ' ' + (ticket.teacher?.middle_name || '');
      const groupName = ticket.group?.group_name || '';
      const cabinetName = ticket.cabinet?.cabinet_name || '';
      const date = new Date(transfers);

      return (searchQuery === teacherFullName && ticket.date === transfers) ||
          (searchQuery === groupName && ticket.date === transfers) ||
          (searchQuery === cabinetName && ticket.date === transfers) ||

          ((searchQuery === groupName && ticket.practice_bool) && date >= new Date(ticket.startpractice) && date <= new Date(ticket.endpractice));
    },
  },
  computed: {
    filteredLesson() {
      return this.lesson.filter(ticket => this.checkVisibility(ticket));
    },
    groupedTickets() {
      const groups = {};
      this.filteredLesson.forEach(ticket => {
        if (!groups[ticket.number.numberlesson_name]) {
          groups[ticket.number.numberlesson_name] = [];
        }
        groups[ticket.number.numberlesson_name].push(ticket);
      });
      return groups;
    },
    // Добавление стилей для практики
    isPractice() {
      return (tickets) => {
        return tickets.some(ticket => ticket.number.numberlesson_name === 'Практика');
      };
    }
  },
  mounted() {
    this.getLessons()
  }
}
</script>

<style>
.v-row {
  display: flex;
  flex-wrap: wrap;
  flex: 1 1 auto;
  margin: 0px;
}
.v-row + .v-row {
  margin-top: 0px;
}
.v-col, .v-col-auto, .v-col-12, .v-col-11, .v-col-10, .v-col-9, .v-col-8, .v-col-7, .v-col-6, .v-col-5, .v-col-4, .v-col-3, .v-col-2, .v-col-1 {
  width: 100%;
  padding: 0px;
}
.v-input--density-default {
  --v-input-control-height: 56px;
  --v-input-padding-top: 18px;
}
.ticket {
  display: grid;
  background: var(--ticket);
  border-radius: 20px;
  height: 90px;
  width: 100%;
  filter: drop-shadow(0px 4px 4px rgba(114, 114, 114, 0.25));
  margin: 10px 0 0;
  transition: all .5s ease-out;
}
.dark .ticket {
  background: var(--ticket-dark);
  filter: none;
  color: var(--dark-text);
}
.ticket-row {
  max-height: 90px;
  min-height: 45px;
}
.ticket .ticket-row:not(:first-child) .ticket-number {
  visibility: hidden;
}
.ticket:active .ticket-time {
  display: flex;
}
.ticket:active .ticket-number-num {
  display: none;
}
.ticket:active .ticket-number {
  background-color: var(--main-color);
  color: var(--dark-text);
}
.dark .ticket:active .ticket-number {
  background-color: var(--dark-main);
}
/*Стили для практики*/
.practice {
  margin-bottom: 30px;
}
.practice:hover .ticket-time {
  display: none!important;
}
.practice:hover .ticket-number-num {
  display: flex!important;
}
.practice .number-num {
  font-size: clamp(15px, 2.5vw, 22px);
}
/**/
.ticket-row div:not(:first-child) {
  font-size: clamp(15px, 1.7vw, 22px);
}
.ticket-time {
  color: var(--dark-text);
  display: none;
  flex-direction: column;
  font-size: 22px;
  font-weight: 700;
}
.ticket-number-num {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}
.number-num {
  font-size: 32px;
  height: 30px;
  display: flex;
  align-items: center;
}
.number-short {
  font-size: 20px;
}
.ticket-number {
  grid-area: ticket-number;
  display: flex;
  height: 90px;
  justify-content: center;
  align-items: center;
  border: 2px solid var(--main-color);
  border-radius: 20px 20px 0px 20px;
  max-width: min-content;
  width: fit-content;
  min-width: 90px;
  margin: 0 18px 0 0;
  padding: 0 12px;
  transition: background-color 0.3s ease;
}
.dark .ticket-number {
  border: 2px solid var(--dark-main);
}
.ticket-theme {
  grid-area: ticket-theme;
  display: flex;
  align-items: center;
  max-height: 90px;
  height: 100%;
  min-height: 45px;
}
.ticket-theme:empty { display: none }
.ticket-teacher {
  grid-area: ticket-teacher;
  display: flex;
  align-items: center;
  max-height: 90px;
  height: 100%;
  min-height: 45px;
}
.ticket-theme:empty ~ .ticket-teacher { display: none }
.ticket-group {
  grid-area: ticket-group;
  display: flex;
  align-items: center;
  max-height: 90px;
  height: 100%;
  min-height: 45px;
}
.ticket-subgroup {
  grid-area: ticket-subgroup;
  display: flex;
  align-items: center;
  max-height: 90px;
  height: 100%;
  min-height: 45px;
}
.ticket-subgroup:empty { display: none }
.ticket-cabinet {
  grid-area: ticket-cabinet;
  display: flex;
  align-items: center;
  max-height: 90px;
  height: 100%;
  min-height: 45px;
}
.ticket-cabinet:empty { display: none }
.lazy {
  display: flex;
  justify-content: center;
  font-size: clamp(15px, 1.7vw, 22px);
  margin-bottom: 60px;
}
.ticket-practice_name {
  grid-area: practice_name;
  display: flex;
  align-items: center;
  max-height: 90px;
  height: 100%;
  min-height: 45px;
}
.ticket-practice_name:empty { display: none }
.ticket-startpractice {
  grid-area: startpractice;
  display: flex;
  align-items: center;
  max-height: 90px;
  height: 100%;
  min-height: 45px;
}
.ticket-startpractice:empty { display: none }
.ticket-endpractice {
  grid-area: endpractice;
  display: flex;
  align-items: center;
  max-height: 90px;
  height: 100%;
  min-height: 45px;
}
.ticket-endpractice:empty { display: none }
</style>
