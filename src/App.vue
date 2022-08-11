<template>
  <div id="app">
    <div class="container">
      <m-calendar :calendar="calendar" />

      <div v-if="parsedSettings.edit" class="editor">
        <textarea
          :class="{ 'editor--broken': settingsBad }"
          v-model="settings"
        />
        <button @click="freeze" type="button">Freeze</button>
      </div>
    </div>
  </div>
</template>

<script>
import MCalendar from './components/MCalendar.vue';

const moment = require('moment');

const defaultSettings = (now) => ({
  partialTitle: null,
  title: null,
  date: null,
  edit: true,
  events: {
    // Place 1
    [`${now.format('yyyy-MM')}-03`]: {
      border: '#FAA6C4',
      color: '#202225',
      start: true,
      text: '🚘',
    },
    [`${now.format('yyyy-MM')}-04`]: {
      border: '#FAA6C4',
      color: '#202225',
      ongoing: true,
      text: '🍻',
    },
    [`${now.format('yyyy-MM')}-05`]: {
      border: '#FAA6C4',
      color: '#202225',
      ongoing: true,
      text: '🍻',
    },
    [`${now.format('yyyy-MM')}-06`]: {
      border: '#FAA6C4',
      color: '#202225',
      ongoing: true,
      text: '🫂',
    },
    [`${now.format('yyyy-MM')}-07`]: {
      border: '#FAA6C4',
      color: '#202225',
      end: true,
      text: '🚘',
    },
    // Place 2
    [`${now.format('yyyy-MM')}-13`]: {
      border: '#3BA55D',
      color: '#202225',
      single: true,
      text: '🥾',
    },
    // Place 3
    [`${now.format('yyyy-MM')}-16`]: {
      border: '#5865F2',
      color: '#202225',
      start: true,
      text: '⛴️',
    },
    [`${now.format('yyyy-MM')}-17`]: {
      border: '#5865F2',
      color: '#202225',
      ongoing: true,
      text: '🫂',
    },
    [`${now.format('yyyy-MM')}-18`]: {
      border: '#5865F2',
      color: '#202225',
      ongoing: true,
      text: '🫂',
    },
    [`${now.format('yyyy-MM')}-19`]: {
      border: '#5865F2',
      color: '#202225',
      end: true,
      text: '⛴️',
    },
  },
});

export default {
  name: 'App',
  components: {
    MCalendar,
  },
  data() {
    const now = moment();

    let settings = '{}';
    try {
      settings = Buffer.from(window.location.hash.replace(/^#/, ''), 'base64').toString('utf8') || '{}';
      JSON.parse(settings);
    } catch (ex) {
      settings = '{}';
      console.error('Ignored error during user-supplied settings parse', ex);
    }
    if (!settings || settings === '{}') {
      settings = JSON.stringify(defaultSettings(now), null, 2);
    }

    let lastGoodSettings;
    let parsedSettings;
    try {
      parsedSettings = JSON.parse(settings);
      if (parsedSettings.date === undefined) {
        throw new Error('Bad Settings');
      }
      lastGoodSettings = settings;
    } catch (ex) {
      parsedSettings = {};
      lastGoodSettings = '{}';
      console.error('Ignored error during user-supplied settings parse and save', ex);
    }

    return {
      now,
      settings,
      lastGoodSettings,
      parsedSettings,
    };
  },
  computed: {
    settingsBad() {
      return this.settings !== this.lastGoodSettings;
    },
    calendar() {
      const settings = {
        ...defaultSettings(this.now),
        ...this.parsedSettings,
      };

      if (!settings.date) {
        settings.date = this.now.format('yyyy-MM-DD');
      }
      const time = moment(settings.date);

      if (!settings.title) {
        settings.title = [
          settings.partialTitle,
          time.format('MMMM yyyy'),
        ].filter((v) => !!v).join(' ');
      }

      if (!settings.events) {
        settings.events = {};
      }

      const data = {
        title: settings.title,
        start: time.clone().startOf('month').startOf('week'),
        end: time.clone().endOf('month').endOf('week'),
        days: [],
        weeks: [],
        activeMonth: time.month(),
      };

      let lastWeek = data.start.week();
      let week = [];
      for (let i = data.start.clone(); !i.isAfter(data.end); i = i.add(1, 'day')) {
        const date = i.clone();
        if (date.week() !== lastWeek) {
          data.weeks.push(week);
          week = [];
          lastWeek = date.week();
        }

        const eventString = date.format('yyyy-MM-DD');
        if (settings.events[eventString]) {
          date.event = settings.events[eventString];
        }

        data.days.push(date);
        week.push(date);
      }

      if (week.length > 0) {
        data.weeks.push(week);
      }

      return data;
    },
  },
  methods: {
    freeze() {
      window.history.pushState(null, null, `#${Buffer.from(this.lastGoodSettings, 'utf8').toString('base64')}`); // save first
      window.history.pushState(null, null, `#${Buffer.from(this.lastGoodSettings, 'utf8').toString('base64')}`); // save first, intentional
      this.settings = JSON.stringify({
        ...this.parsedSettings,
        edit: false,
      }); // intentionally minified
    },
  },
  watch: {
    settings() {
      try {
        const parsedSettings = JSON.parse(this.settings);
        if (parsedSettings.date === undefined) {
          throw new Error('Bad Settings');
        }
        this.parsedSettings = parsedSettings;
        this.lastGoodSettings = this.settings;
        window.history.replaceState(null, null, `#${Buffer.from(this.lastGoodSettings, 'utf8').toString('base64')}`);
      } catch (ex) {
        // do nothing
        console.error('Ignored error during user-supplied settings parse on watch', ex);
      }
    },
  },
};
</script>

<style lang="scss">
html, body {
  color: #D6D7D8;
  background-color: #36393F;
  font-family: sans-serif;
}
</style>

<style lang="scss" scoped>
#app {
  display: flex;
}

.container {
  margin: 0 auto;
}

.editor {
  padding-top: 5em;
  display: flex;
  flex-direction: column;
  width: 100%;
  max-width: 800px;

  border: 3px solid transparent;
  &--broken {
    border-color: red;
  }

  textarea {
    height: 500px;
    color: #D6D7D8;
    background-color: #36393F;
  }
}

</style>
