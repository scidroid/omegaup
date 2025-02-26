<template>
  <div>
    <div class="text-center mb-4">
      <h2>
        {{ T.submissionsListTitle }}
      </h2>
      <h4 v-if="!includeUser && submissions.length > 0">
        {{ T.wordsBy }}
        <omegaup-username
          :username="submissions[0].username"
          :classname="submissions[0].classname"
          :linkify="true"
        ></omegaup-username>
      </h4>
    </div>
    <div class="card">
      <h5 class="card-header">
        {{
          ui.formatString(T.submissionsRangeHeader, {
            lowCount: (page - 1) * length + 1,
            highCount: page * length,
          })
        }}
      </h5>
      <div v-if="includeUser" class="card-body d-flex align-items-center">
        <omegaup-common-typeahead
          :existing-options="searchResultUsers"
          :value.sync="searchedUsername"
          :max-results="10"
          class="mr-2"
          @update-existing-options="
            (query) => $emit('update-search-result-users', query)
          "
        />

        <a :href="`/submissions/${encodeURIComponent(searchedUsername)}/`">
          <button class="btn btn-primary" type="button">
            {{ T.searchUser }}
          </button>
        </a>
      </div>
      <div class="table-responsive">
        <table class="table mb-0 submissions-table">
          <thead>
            <tr>
              <th scope="col" class="text-center">{{ T.wordsTime }}</th>
              <th v-if="includeUser" scope="col" class="text-center">
                {{ T.wordsUser }}
              </th>
              <th scope="col" class="text-center">{{ T.wordsProblem }}</th>
              <th
                :class="{ 'fixed-width-column': includeUser }"
                class="text-center"
                scope="col"
              >
                {{ T.wordsLanguage }}
              </th>
              <th scope="col" class="text-center fixed-with-column">
                {{ T.wordsVerdict }}
              </th>
              <th scope="col" class="text-right">{{ T.wordsRuntime }}</th>
              <th scope="col" class="text-right">{{ T.wordsMemory }}</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(submission, index) in submissions" :key="index">
              <td class="text-center">
                {{ time.formatDateTime(submission.time) }}
              </td>
              <td v-if="includeUser" class="text-center">
                <omegaup-username
                  :username="submission.username"
                  :classname="submission.classname"
                  :linkify="true"
                >
                </omegaup-username>
                <br />
                <a
                  class="school-text"
                  :href="`/schools/profile/${submission.school_id}/`"
                  >{{ submission.school_name }}</a
                >
              </td>
              <td class="text-center">
                <a :href="`/arena/problem/${submission.alias}/`">{{
                  submission.title
                }}</a>
              </td>
              <td class="text-center">{{ submission.language }}</td>
              <td
                class="text-center verdict"
                :class="`verdict-${submission.verdict}`"
              >
                {{ T[`verdict${submission.verdict}`] }}
              </td>
              <td class="text-right">
                {{
                  submission.runtime === 0
                    ? '—'
                    : ui.formatString(T.submissionRunTimeInSeconds, {
                        value: (
                          parseFloat(submission.runtime || '0') / 1000
                        ).toFixed(2),
                      })
                }}
              </td>
              <td class="text-right">
                {{
                  submission.memory === 0
                    ? '—'
                    : ui.formatString(T.submissionMemoryInMegabytes, {
                        value: (
                          parseFloat(submission.memory) /
                          (1024 * 1024)
                        ).toFixed(2),
                      })
                }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="card-footer">
        <omegaup-common-paginator
          :pager-items="pagerItems"
        ></omegaup-common-paginator>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Vue, Component, Prop } from 'vue-property-decorator';
import { types } from '../../api_types';
import T from '../../lang';
import * as ui from '../../ui';
import * as time from '../../time';
import UserName from '../user/Username.vue';
import common_Typeahead from '../common/Typeahead.vue';
import common_Paginator from '../common/Paginator.vue';

@Component({
  components: {
    'omegaup-username': UserName,
    'omegaup-common-typeahead': common_Typeahead,
    'omegaup-common-paginator': common_Paginator,
  },
})
export default class SubmissionsList extends Vue {
  @Prop() page!: number;
  @Prop() length!: number;
  @Prop() includeUser!: boolean;
  @Prop() totalRows!: number;
  @Prop() submissions!: types.Submission[];
  @Prop() pagerItems!: types.PageItem[];
  @Prop() searchResultUsers!: types.ListItem[];

  T = T;
  ui = ui;
  time = time;
  searchedUsername: null | string = null;

  get showNextPage(): boolean {
    return this.length * this.page < this.totalRows;
  }

  get showControls(): boolean {
    return this.showNextPage || this.page > 1;
  }
}
</script>

<style lang="scss" scoped>
@import '../../../../sass/main.scss';
table.submissions-table > tbody > tr > td {
  vertical-align: middle;
}

.verdict-AC {
  background: var(--arena-submissions-list-verdict-ac-background-color);
}

.verdict-CE {
  background: --arena-submissions-list-verdict-ce-background-color;
}

.verdict-JE,
.verdict-VE {
  background: var(--arena-submissions-list-verdict-je-ve-background-color);
}

.school-text {
  font-size: 0.9em;
}

.fixed-width-column {
  width: 180px;
}
</style>
