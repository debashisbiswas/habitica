<template>
  <div>
    <div
      class="row"
    >
      <div
        v-if="quest.collect"
        class="table-row"
      >
        <dt>{{ $t('collect') + ':' }}</dt>
        <dd>
          <div
            v-for="(collect, key) of quest.collect"
            :key="key"
          >
            <span>{{ collect.count }} {{ getCollectText(collect) }}</span>
          </div>
        </dd>
      </div>
      <div
        v-if="quest.boss"
        class="table-row"
      >
        <dt>{{ $t('bossHP') + ':' }}</dt>
        <dd>{{ quest.boss.hp }}</dd>
      </div>
      <div class="table-row">
        <dt>{{ $t('difficulty') + ':' }}</dt>
        <dd>
          <div
            v-for="(star, index) of stars()"
            :key="index"
            class="svg-icon inline icon-16"
            v-html="icons[star]"
          ></div>
        </dd>
      </div>
    </div>
    <div v-if="quest.event">
      {{ limitedString }}
    </div>
  </div>
</template>

<style lang="scss" scoped>
@import '~@/assets/scss/colors.scss';

.row {
  display: table;
  margin: 0;
}

.table-row {
  display: table-row;
  margin-bottom: 4px;
}

dd {
  height: 24px;
  padding-left: 1em;
  padding-top: 3px;
  padding-bottom: 3px;
}

dt, dd {
  display: table-cell;
  vertical-align: middle;
}

dt, dd, dd > * {
  text-align: left;
}

dt {
  font-size: 1.3em;
  line-height: 1.2;
  color: $gray-50;
}

.svg-icon {
  margin-right: 4px;
}

.small-version {
  font-size: 12px;
  line-height: 1.33;

  .svg-icon {
    margin-top: 1px;
  }
}
</style>

<style lang="scss">
.questPopover {
  dt {
    color: inherit;
    font-size: 1em;
    white-space: nowrap;
  }
}

// popover used in quest selection modal
.popover-body {
  dt {
    color: inherit;
  }
}

// making sure the star-colors always correct
.star {
  fill: #ffb445;
}
.star-empty {
  fill: #686274;
}

</style>

<script>
import moment from 'moment';

import svgStar from '@/assets/svg/difficulty-star.svg';
import svgStarHalf from '@/assets/svg/difficulty-star-half.svg';
import svgStarEmpty from '@/assets/svg/difficulty-star-empty.svg';

import seasonalShopConfig from '@/../../common/script/libs/shops-seasonal.config';

export default {
  props: {
    quest: {
      type: Object,
    },
  },
  data () {
    return {
      icons: Object.freeze({
        star: svgStar,
        starHalf: svgStarHalf,
        starEmpty: svgStarEmpty,
      }),
      timer: '',
      limitedString: '',
    };
  },
  computed: {
    difficulty () {
      if (this.quest.boss) {
        return this.quest.boss.str;
      }

      return 1;
    },
  },
  mounted () {
    this.countdownString();
    this.timer = setInterval(this.countdownString, 1000);
  },
  methods: {
    stars () {
      const result = [];
      const { difficulty } = this;

      for (let i = 1; i <= 4; i += 1) {
        const diff = difficulty - i;

        if (diff >= 0) {
          result.push('star');
        } else if (diff <= -1) {
          result.push('starEmpty');
        } else {
          result.push('starHalf');
        }
      }

      return result;
    },
    getCollectText (collect) {
      if (collect.text instanceof Function) {
        return collect.text();
      }
      return collect.text;
    },
    countdownString () {
      const diffDuration = moment.duration(moment(seasonalShopConfig.dateRange.end).diff(moment()));

      if (diffDuration.asSeconds() <= 0) {
        this.limitedString = this.$t('noLongerAvailable');
      } else if (diffDuration.days() > 0) {
        this.limitedString = this.$t('limitedAvailabilityDays', {
          days: diffDuration.days(),
          hours: diffDuration.hours(),
          minutes: diffDuration.minutes(),
        });
      } else if (diffDuration.asMinutes() > 2) {
        this.limitedString = this.$t('limitedAvailabilityHours', {
          hours: diffDuration.hours(),
          minutes: diffDuration.minutes(),
        });
      } else {
        this.limitedString = this.$t('limitedAvailabilityMinutes', {
          minutes: diffDuration.minutes(),
          seconds: diffDuration.seconds(),
        });
      }
    },
    cancelAutoUpdate () {
      clearInterval(this.timer);
    },
  },
  beforeDestroy () {
    this.cancelAutoUpdate();
  },
};
</script>
