<template>

  <CoreBase
    :immersivePage="false"
    :authorized="userIsAuthorized"
    authorizedRole="adminOrCoach"
    :showSubNav="true"
  >

    <template #sub-nav>
      <TopNavbar />
    </template>

    <KPageContainer>

      <ReportsGroupReportQuizHeader />

      <ReportsControls @export="exportCSV" />

      <CoreTable :emptyMessage="coachString('questionListEmptyState')">
        <template #headers>
          <th>{{ coachString('questionLabel') }}</th>
          <th>{{ coachString('helpNeededLabel') }}</th>
        </template>
        <template #tbody>
          <transition-group tag="tbody" name="list">
            <tr v-for="tableRow in table" :key="tableRow.question_id">
              <td>
                <KRouterLink
                  :text="tableRow.title"
                  :to="questionLink(tableRow.question_id)"
                  icon="question"
                />
              </td>
              <td>
                <LearnerProgressRatio
                  :verb="VERBS.needHelp"
                  :icon="ICONS.help"
                  :total="tableRow.total"
                  :count="tableRow.total - tableRow.correct"
                  :verbosity="1"
                />
              </td>
            </tr>
          </transition-group>
        </template>
      </CoreTable>
    </KPageContainer>
  </CoreBase>

</template>


<script>

  import { mapGetters } from 'vuex';
  import commonCoach from '../common';
  import LearnerProgressRatio from '../common/status/LearnerProgressRatio';
  import CSVExporter from '../../csv/exporter';
  import * as csvFields from '../../csv/fields';
  import ReportsGroupReportQuizHeader from './ReportsGroupReportQuizHeader';
  import ReportsControls from './ReportsControls';
  import { PageNames } from './../../constants';

  export default {
    name: 'ReportsGroupReportQuizQuestionListPage',
    components: {
      ReportsGroupReportQuizHeader,
      ReportsControls,
      LearnerProgressRatio,
    },
    mixins: [commonCoach],
    computed: {
      ...mapGetters('questionList', ['difficultQuestions']),
      exam() {
        return this.examMap[this.$route.params.quizId];
      },
      group() {
        return this.groupMap[this.$route.params.groupId];
      },
      table() {
        return this.difficultQuestions.map(question => {
          const tableRow = {};
          Object.assign(tableRow, question);
          return tableRow;
        });
      },
    },
    methods: {
      questionLink(questionId) {
        return this.classRoute(PageNames.REPORTS_GROUP_REPORT_QUIZ_QUESTION_PAGE_ROOT, {
          questionId,
          quizId: this.$route.params.quizId,
        });
      },
      exportCSV() {
        const columns = [
          {
            name: this.coachString('questionLabel'),
            key: 'title',
          },
          ...csvFields.helpNeeded(),
        ];

        const exporter = new CSVExporter(columns, this.className);
        exporter.addNames({
          group: this.group.name,
          resource: this.exam.title,
          difficultQuestions: this.coachString('difficultQuestionsLabel'),
        });

        exporter.export(this.table);
      },
    },
  };

</script>


<style lang="scss" scoped>

  @import '../common/print-table';

  .stats {
    margin-right: 16px;
  }

</style>
