<template>
  <section :class="{ 'search-grid': true, 'search-grid__contain-images': shouldContainImages }"
           ref="searchGrid">
    <div class="search-grid_ctr" ref="gridItems">
      <div v-show="!isFetchingImages && includeAnalytics" class="search-grid_analytics count">
        <h2>{{ searchTerm }}</h2>
        <span class="caption has-text-weight-semibold"> {{ _imagesCount }}</span>
        <div class="is-pulled-right padding-right-big is-hidden-touch">
          <search-rating v-if="_query.q" :searchTerm="_query.q" />
        </div>
        <div class="is-hidden-desktop is-block">
          <search-rating v-if="_query.q" :searchTerm="searchTerm" />
        </div>
      </div>
      <div class="search-grid-cells">
        <search-grid-cell
          v-for="(image, index) in _images" :key="index" :image="image">
        </search-grid-cell>
      </div>
      <div class="load-more">
        <button v-show="!isFetchingImages && includeAnalytics"
                class="button margin-bottom-big"
                :disabled="isFinished"
                @click="onLoadMoreImages">
          <span v-if="isFinished">No more images :(</span>
          <span v-else>Load more results</span>
        </button>
        <loading-icon v-show="isFetchingImages" />
      </div>
      <div class="search-grid_notification callout alert" v-if="isFetchingImagesError">
        <h5>Error fetching images: {{_errorMessage }}</h5>
      </div>
    </div>
  </section>
</template>

<script>
import { SET_IMAGES } from '@/store/mutation-types';
import SearchGridCell from '@/components/SearchGridCell';
import LoadingIcon from '@/components/LoadingIcon';
import SearchRating from '@/components/SearchRating';

const DEFAULT_PAGE_SIZE = 20;

export default {
  name: 'search-grid-manual-load',
  components: {
    SearchGridCell,
    LoadingIcon,
    SearchRating,
  },
  data: () => ({
    isDataInitialized: false,
    shouldContainImages: false,
  }),
  props: {
    imagesCount: {
      default: 0,
    },
    images: {
      default: () => ([]),
    },
    query: {},
    useInfiniteScroll: {
      default: true,
    },
    includeAnalytics: {
      default: true,
    },
    includeAddToList: {
      default: true,
    },
    searchTerm: {
      default: '',
    },
  },
  computed: {
    imagePage() {
      return this.$store.state.imagePage;
    },
    isFetchingImagesError() {
      return this.$store.state.isFetchingImagesError;
    },
    isFetchingImages() {
      return this.$store.state.isFetchingImages;
    },
    _images() {
      return this.useInfiniteScroll ? this.$store.state.images : this.images;
    },
    currentPage() {
      return this.$store.state.imagePage;
    },
    _imagesCount() {
      const count = this.useInfiniteScroll ? this.$store.state.imagesCount : this.imagesCount;
      return count >= 10000 ? `Over ${count.toLocaleString('en')} images` : `${count.toLocaleString('en')} images`;
    },
    _query() {
      return this.$props.query;
    },
    _errorMessage() {
      return this.$store.state.errorMsg;
    },
    isFinished() {
      return this.currentPage >= this.$store.state.pageCount;
    },
  },
  watch: {
    _images: {
      handler() {
        if (this.$state) {
          this.$state.loaded();
          if (this._imagesCount < this.currentPage * DEFAULT_PAGE_SIZE) {
            this.$state.complete();
          }
        }
        this.isDataInitialized = true;
      },
    },
    _query: {
      handler() {
        this.searchChanged();
      },
      deep: true,
    },
  },
  methods: {
    handleScalingChange() {
      setTimeout(() => {
        this.$redrawVueMasonry(); // Some elements end up taking less space
      }, 100); // One-tenth of a second should be sufficient to calculate new height
    },
    searchChanged() {
      this.$store.commit(SET_IMAGES, { images: [], page: 1 });
    },
    onLoadMoreImages() {
      const searchParams = {
        page: this.currentPage + 1,
        shouldPersistImages: true,
        ...this._query,
      };

      this.$emit('onLoadMoreImages', searchParams);
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
  @import "node_modules/bulma/sass/utilities/_all";

  .button[disabled] {
    opacity: 1;
  }

  .search-grid_analytics h5,
  .search-grid_layout-control h5 {
    padding-top: 1.36vh;
    font-size: 1rem;
    display: inline-block;
  }

  .search-grid_layout-control h5 {
    margin-right: 10px;
  }

  .search-grid_layout-control {
    text-align: right;

    fieldset {
      display: inline;
      margin-right: 5px;
    }
  }

  .infinite-loading-container {
    margin-top: 30px;
    width: 100%;
  }

  .search-grid_ctr {
    .masonry-layout {
      margin: auto;
      margin-top: 1rem;
    }

    .item {
      width: 320px;
      margin-bottom: 20px;
    }
  }

  .search-grid:after {
    content: '';
    display: block;
    clear: both;
  }

  .search-grid_notification {
    width: 50%;
    margin: auto;
    font-weight: 500;
    text-align: center;
  }

  .search-grid-cells {
    display: flex;
    flex-wrap: wrap;
    margin: 10px;

    @include tablet {
      &:after {
        content: '';
        flex-grow: 999999999;
      }
    }
  }

  label {
    color: #333333;
  }

  .load-more {
    text-align: center;

    button {
      font-size: 1.2em;

      @include mobile {
        padding: .5rem;

        span{
          font-size: 0.9rem;
        }
      }
    }
  }
  .count{
    padding-top: 0.6rem;
    padding-left: 1.3rem;
  }
</style>
