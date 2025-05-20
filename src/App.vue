<template>
  <div class="availability-container">
    <div class="scroll-wrapper">
      <!-- Week Number Row -->
      <div
        class="grid-row week-number-row"
        :style="{ gridTemplateColumns: gridColumns }"
      >
        <div class="cell placeholder" v-for="i in 4" :key="'ph-' + i"></div>
        <template v-for="(week, index) in weekLabels" :key="'week-' + index">
          <div
            class="cell week-span"
            :style="{ gridColumn: 'span ' + computeSpan(index) }"
          >
            {{ week.label }}
          </div>
        </template>
        <div class="cell placeholder"></div>
      </div>

      <!-- Header Row -->
      <div
        class="grid-row header-row"
        :style="{ gridTemplateColumns: gridColumns }"
      >
        <div class="cell sticky-left sticky-1">Sprint participant</div>
        <div class="cell sticky-left sticky-2">Baseline SP</div>
        <div class="cell sticky-left sticky-3">Note</div>
        <div class="cell sticky-left sticky-4">PDO</div>

        <div
          v-for="(date, index) in dateHeaders"
          :key="'header-' + index"
          class="cell date-cell"
          :class="{ today: isToday(date) }"
        >
          {{ format(date, "E d") }}
        </div>

        <div class="cell sticky-right">Available SP</div>
      </div>

      <!-- Participant Rows -->
      <div
        v-for="(participant, pIndex) in participants"
        :key="'row-' + pIndex"
        class="grid-row"
        :style="{ gridTemplateColumns: gridColumns }"
      >
        <div class="cell sticky-left sticky-1 participant">
          <img :src="participant.avatar" class="avatar" />
          {{ participant.name }}
        </div>
        <div class="cell sticky-left sticky-2">
          <input v-model.number="participant.baselineSP" type="number" />
        </div>
        <div class="cell sticky-left sticky-3">💬</div>
        <div class="cell sticky-left sticky-4">0</div>

        <div
          v-for="(date, dIndex) in dateHeaders"
          :key="'cell-' + pIndex + '-' + dIndex"
          class="cell date-cell"
          @click="toggleDayOff(pIndex, dIndex)"
        >
          <span v-if="participant.daysOff.includes(dIndex)">✖</span>
        </div>

        <div class="cell sticky-right">
          {{ computeAvailableSP(participant) }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {
  addDays,
  format,
  startOfDay,
  differenceInCalendarDays,
  getISOWeek,
  isToday,
} from "date-fns";

export default {
  name: "SprintAvailability",
  data() {
    return {
      startDate: startOfDay(new Date(2025, 0, 6)), // Jan 6, 2025
      endDate: startOfDay(new Date(2025, 1, 7)), // Feb 7, 2025
      participants: [
        {
          name: "John Doe",
          avatar: "https://i.pravatar.cc/40?img=1",
          baselineSP: 10,
          daysOff: [],
        },
        {
          name: "Jill Valentine",
          avatar: "https://i.pravatar.cc/40?img=2",
          baselineSP: 10,
          daysOff: [],
        },
        {
          name: "Jeremiah Sora",
          avatar: "https://i.pravatar.cc/40?img=3",
          baselineSP: 10,
          daysOff: [],
        },
        {
          name: "Jim Halpert",
          avatar: "https://i.pravatar.cc/40?img=4",
          baselineSP: 10,
          daysOff: [],
        },
        {
          name: "Jan Oda",
          avatar: "https://i.pravatar.cc/40?img=5",
          baselineSP: 10,
          daysOff: [],
        },
      ],
    };
  },
  computed: {
    totalDays() {
      return differenceInCalendarDays(this.endDate, this.startDate) + 1;
    },
    dateHeaders() {
      return Array.from({ length: this.totalDays }, (_, i) =>
        addDays(this.startDate, i)
      );
    },
    weekLabels() {
      const weeks = [];
      let i = 0;
      while (true) {
        const dayIndex = i * 7;
        const date = addDays(this.startDate, dayIndex);
        if (date > this.endDate) break;
        weeks.push({ label: `Week ${getISOWeek(date)}`, spanStart: dayIndex });
        i++;
      }
      return weeks;
    },
    gridColumns() {
      return `200px 100px 60px 60px repeat(${this.totalDays}, 44px) 100px`;
    },
  },
  methods: {
    format,
    isToday,
    toggleDayOff(participantIndex, dayIndex) {
      const daysOff = this.participants[participantIndex].daysOff;
      const i = daysOff.indexOf(dayIndex);
      if (i === -1) daysOff.push(dayIndex);
      else daysOff.splice(i, 1);
    },
    computeAvailableSP(participant) {
      return (
        participant.baselineSP -
        participant.daysOff.length * 0.1
      ).toFixed(1);
    },
    computeSpan(index) {
      const start = this.weekLabels[index].spanStart;
      const remaining = this.totalDays - start;
      return Math.min(7, remaining);
    },
  },
};
</script>

<style scoped>
.availability-container {
  font-family: Arial, sans-serif;
  overflow-x: auto;
}

.scroll-wrapper {
  overflow-x: auto;
  position: relative;
}

.grid-row {
  display: grid;
  align-items: center;
  min-width: max-content;
}

.header-row {
  font-weight: bold;
  background-color: #f5f5f5;
  position: sticky;
  top: 0;
  z-index: 3;
}

.week-number-row {
  background: #f9f9f9;
  font-weight: bold;
}

.week-span {
  text-align: center;
  padding: 8px 0;
  background: #f0f0ff;
}

.placeholder {
  background: #f9f9f9;
}

.cell {
  padding: 8px;
  text-align: center;
  background-color: white;
  white-space: nowrap;
}

.date-cell {
  cursor: pointer;
  min-width: 40px;
}

.today {
  background-color: #ffecec;
}

.sticky-left {
  position: sticky;
  background-color: #fff;
  z-index: 2;
}

.sticky-1 {
  left: 0;
  z-index: 3;
}
.sticky-2 {
  left: 200px;
  z-index: 3;
}
.sticky-3 {
  left: 300px;
  z-index: 3;
}
.sticky-4 {
  left: 360px;
  z-index: 3;
}

.sticky-right {
  position: sticky;
  right: 0;
  background-color: #fff;
  z-index: 2;
}

.participant {
  display: flex;
  align-items: center;
  gap: 6px;
}

.avatar {
  width: 24px;
  height: 24px;
  border-radius: 50%;
}

input[type="number"] {
  width: 60px;
  padding: 4px;
}
</style>
