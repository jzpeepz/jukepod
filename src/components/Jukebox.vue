<template>
  <div class="container flex flex-col items-center px-8">
    <div class="relative" style="width: 60%;">
      <select
        v-model="selectedPodcast"
        @change="changeFeed()"
        class="block appearance-none w-full bg-gray-200 border border-gray-200 text-gray-700 py-3 px-4 pr-8 rounded leading-tight focus:outline-none focus:bg-white focus:border-gray-500"
      >
        <option value="" disabled selected>Choose Podcast</option>
        <option
          :value="podcast.url"
          v-for="podcast in podcasts"
          :key="podcast.url"
          v-html="podcast.name"
        ></option>
      </select>
      <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center px-2 text-gray-700">
        <svg class="fill-current h-4 w-4" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20"><path d="M9.293 12.95l.707.707L15.657 8l-1.414-1.414L10 10.828 5.757 6.586 4.343 8z"/></svg>
      </div>
    </div>

      <article v-show="item.title.length > 0">
        <h1 class="text-4xl my-6" v-html="item.title"></h1>
        <figure>
            <figcaption v-html="item.summary"></figcaption>
            <audio
                ref="audio"
                controls
                preload="metadata"
                :src="item.url"
                @timeupdate="timestampUpdate"
                class="outline-none ml-auto mr-auto mb-8"
            >
                    Your browser does not support the <code>audio</code> element.
            </audio>
        </figure>
        <div class="flex justify-center">
          <button
            @click="next()"
            style=""
            class="bg-gray-200 hover:bg-gray-400 text-gray-800 font-bold py-2 pl-4 pr-3 rounded flex items-center uppercase"
          >
            <span class="leading-none">Next</span>
            <svg aria-hidden="true" xmlns="http://www.w3.org/2000/svg" class="fill-current w-3 h-3 ml-1" viewBox="0 0 256 512">
              <path fill="currentColor" d="M17.525 36.465l-7.071 7.07c-4.686 4.686-4.686 12.284 0 16.971L205.947 256 10.454 451.494c-4.686 4.686-4.686 12.284 0 16.971l7.071 7.07c4.686 4.686 12.284 4.686 16.97 0l211.051-211.05c4.686-4.686 4.686-12.284 0-16.971L34.495 36.465c-4.686-4.687-12.284-4.687-16.97 0z"/>
            </svg>
          </button>
        </div>
      </article>
  </div>
</template>

<script>
export default {
  data() {
    return {
      selectedPodcast: '',
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
</style>
