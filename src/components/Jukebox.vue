<template>
  <div class="container">
    <select v-model="selectedPodcast" @change="changeFeed()">
      <option
        :value="podcast.url"
        v-for="podcast in podcasts"
        :key="podcast.url"
        v-html="podcast.name"
      ></option>
    </select>

      <article v-show="item.title.length > 0">
        <h1 v-html="item.title"></h1>
        <figure>
            <figcaption v-html="item.summary"></figcaption>
            <audio
                ref="audio"
                controls
                preload="none"
                :src="item.url"
                @timeupdate="timestampUpdate">
                    Your browser does not support the <code>audio</code> element.
            </audio>
        </figure>
        <button @click="next()">Next</button>
      </article>
  </div>
</template>

<script>
export default {
  data() {
    return {
      selectedPodcast: null,
      podcasts: [
        { name: 'Dateline', url: 'https://rss.art19.com/dateline-nbc' },
        { name: 'Doughboys', url: 'https://rss.art19.com/doughboys' },
      ],
      item: {
        title: '', summary: '', url: '', guid: '',
      },
      items: [],
    };
  },
  mounted() {
    const url = window.localStorage.getItem('juke_podcast_url');
    if (url !== null && url.length > 0) {
      this.selectedPodcast = url;
      this.load();
      this.timestampSet();
    }
  },
  methods: {
    reset() {
      window.localStorage.removeItem('juke_guid');
      window.localStorage.removeItem('juke_timestamp');
    },
    next() {
      const nextItem = this.items[Math.floor(Math.random() * this.items.length)];
      this.item = nextItem;
      this.$refs.audio.play();
      window.localStorage.setItem('juke_guid', nextItem.guid);
    },
    timestampUpdate(event) {
      window.localStorage.setItem('juke_timestamp', event.target.currentTime);
    },
    timestampSet() {
      const timestamp = window.localStorage.getItem('juke_timestamp');

      if (timestamp > 0) {
        this.$refs.audio.currentTime = timestamp;
      }
    },
    changeFeed() {
      this.reset();
      window.localStorage.setItem('juke_podcast_url', this.selectedPodcast);
      this.load();
    },
    load() {
      return fetch(this.selectedPodcast)
        .then((response) => response.text())
        .then((str) => new window.DOMParser().parseFromString(str, 'text/xml'))
        .then((data) => {
          const items = data.querySelectorAll('item');
          return Array.from(items)
            .filter((el) => el.querySelector('*|episodeType').innerHTML === 'full')
            .map((el) => {
              // const summary = el.querySelector('description').innerHTML;
              const summaryText = el.querySelector('*|summary').innerHTML;
              return {
                title: el.querySelector('title').innerHTML,
                summary: summaryText,
                // summary: summary.replace('<![CDATA[', '').replace(']]>', ''),
                url: el.querySelector('enclosure').getAttribute('url'),
                guid: el.querySelector('guid').innerHTML,
              };
            });
        })
        .then((podcasts) => {
          this.items = podcasts;
          let guid = window.localStorage.getItem('juke_guid');
          let item = null;

          if (guid === null || guid.length === 0) {
            // grab a random podcast from the feed
            item = podcasts[Math.floor(Math.random() * podcasts.length)];
            guid = item.guid;
          } else {
            // find podcasts by guid
            item = podcasts.find((element) => element.guid === guid);
          }

          // store the guid of the current podcast for later loading
          window.localStorage.setItem('juke_guid', guid);

          this.item = item;
        });
    },
  },
};
</script>

<style lang="scss" scoped>
.container {
  max-width: 500px;
  margin-left: auto;
  margin-right: auto;
}
figure audio {
  margin-top: 20px;
}
select, button {
  width: 60%;
}

button {
    box-sizing: border-box;
    border: 0 solid #e2e8f0 /* 2 */;
    font-family: inherit;
    font-size: 100%;
    line-height: 1.15;
    margin: 0 /* 2 */;
    overflow: visible;
    text-transform: none;
    -webkit-appearance: button;
    background-color: transparent;
    background-image: none;
    padding: 0;
    cursor: pointer;
    padding: 0;
    line-height: inherit;
    color: inherit;
    --bg-opacity: 1!important;
    background-color: #e2e8f0!important;
    background-color: rgba(226,232,240,var(--bg-opacity))!important;
    border-radius: .25rem!important;
    font-weight: 700!important;
    padding-top: .5rem!important;
    padding-bottom: .5rem!important;
    padding-left: 1rem!important;
    padding-right: 1rem!important;
    --text-opacity: 1!important;
    color: #2d3748!important;
    color: rgba(45,55,72,var(--text-opacity))!important;
}

select {
    font-family: inherit;
    font-size: 100%;
    line-height: 1.15;
    margin: 0 /* 2 */;
    text-transform: none;
    -webkit-appearance: none!important;
    -moz-appearance: none!important;
    appearance: none!important;
    --bg-opacity: 1!important;
    background-color: #edf2f7!important;
    background-color: rgba(237,242,247,var(--bg-opacity))!important;
    --border-opacity: 1!important;
    border-color: #edf2f7!important;
    border-color: rgba(237,242,247,var(--border-opacity))!important;
    border-radius: .25rem!important;
    border-width: 1px!important;
    display: block!important;
    line-height: 1.25!important;
    padding-top: .75rem!important;
    padding-bottom: .75rem!important;
    padding-left: 1rem!important;
    padding-right: 2rem!important;
    --text-opacity: 1!important;
    color: #4a5568!important;
    color: rgba(74,85,104,var(--text-opacity))!important;
    margin-left: auto;
    margin-right: auto;
}
</style>
