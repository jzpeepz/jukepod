<template>
  <div>
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
      // audio.setAttribute('src', nextItem.url);
      this.$refs.audio.play();

      // document.querySelector('#juke-title').innerHTML = nextItem.title;
      // document.querySelector('#juke-summary').innerHTML = nextItem.summary;
      window.localStorage.setItem('juke_guid', nextItem.guid);
    },
    timestampUpdate(event) {
      window.localStorage.setItem('juke_timestamp', event.target.currentTime);
    },
    timestampSet() {
      // const audio = document.querySelector('audio');
      // console.log(audio);
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
              const summary = el.querySelector('description').innerHTML;
              return {
                title: el.querySelector('title').innerHTML,
                // summary: el.querySelector('*|summary').innerHTML,
                summary: summary.replace('<![CDATA[', '').replace(']]>', ''),
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

<style>

</style>
