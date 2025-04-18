<template>
  <div class="d-flex flex-column flex-grow-1">
    <div class="course-search">
      <b-form @submit.prevent="performSearch">
        <b-form-group label="Search" label-for="search">
          <b-form-input
            id="search"
            v-model="textSearch"
            placeholder="Intro to College - COLG 1030"
            list="list-id"
          ></b-form-input>
        </b-form-group>

        <b-row>
          <b-col v-if="subsemesterOptions.length > 2">
            <b-form-group label="Filter Sub-Semester" for="sub-semester">
              <b-form-select
                v-model="selectedSubsemester"
                :options="subsemesterOptions"
              ></b-form-select>
            </b-form-group>
          </b-col>
          <b-col>
            <b-form-group label="Filter Department" for="department">
              <b-form-select
                v-model="selectedDepartment"
                :options="departmentOptions"
              ></b-form-select>
            </b-form-group>
          </b-col>
        </b-row>
        
        <b-button type="submit" variant="primary" class="mt-2" data-cy="search-courses-btn">
          Search Courses
        </b-button>
      </b-form>
    </div>

    <hr />
    <div id="scroll-box" data-cy="course-list" v-if="hasSearched">
      <div v-if="filterCourses.length == 0" class="no-courses">
        Oops, no results!
      </div>
      <DynamicScroller
        v-else
        class="scroller"
        :items="filterCourses"
        :min-item-size="10"
        typeField="vscrl_type"
      >
        <template v-slot="{ item: course, index, active }">
          <DynamicScrollerItem
            :item="course"
            :active="active"
            :size-dependencies="[course.title]"
            :data-index="index"
            :emitResize="true"
          >
            <div
              class="course-listing"
              :class="{ 'bg-light': course.selected }"
            >
              <CourseListing
                :course="course"
                defaultAction="toggleCourse"
                v-on="$listeners"
                lazyLoadCollapse
              >
                <template #toggleCollapseButton="{ course }">
                  <button
                    v-show="
                      course.corequisites ||
                      course.prerequisites ||
                      course.raw_precoreqs
                    "
                    class="btn"
                    @click.stop="courseInfoModalToggle(course)"
                    data-cy="course-info-button"
                  >
                    <font-awesome-icon :icon="faInfoCircle" />
                  </button>
                </template>
                <template #collapseContent>
                  {{ null }}
                </template>
              </CourseListing>
            </div>
          </DynamicScrollerItem>
        </template>
      </DynamicScroller>
    </div>
  </div>
</template>

<script>
import "@/typedef";
import { mapState } from "vuex";
import { faInfoCircle } from "@fortawesome/free-solid-svg-icons";

import { DAY_SHORTNAMES } from "@/utils";

import { getCourses, getDepartments } from "@/services/YacsService";

import CourseListingComponent from "@/components/CourseListing";

import { DynamicScroller, DynamicScrollerItem } from "vue-virtual-scroller";

export default {
  name: "CourseList",
  components: {
    CourseListing: CourseListingComponent,
    DynamicScroller,
    DynamicScrollerItem,
  },
  data() {
    return {
      faInfoCircle,
      DAY_SHORTNAMES,
      textSearch: "",
      selectedSubsemester: null,
      selectedDepartment: null,
      courseList: null,
      hasSearched: false, // New flag to track if search has been performed
    };
  },
  created() {
    getDepartments().then((departments) => {
      this.departmentOptions.push(...departments.map((d) => d.department));
    });
  },
  methods: {
    courseInfoModalToggle(course) {
      this.$emit("showCourseInfo", course);
    },
    /* wrapper for querying with search */
    // todo: get courses should be changed
    //text parameter comes from watch
    performSearch() {
      this.hasSearched = true;
      this.updateCourseList();
    },
    updateCourseList() {
      getCourses(this.selectedSemester, this.textSearch, false).then(
        (course_list) => {
          this.courseList = course_list;
        }
      );
    },
    checkFunction(courseInput, textSearch) {
      const text = textSearch
        .trim()
        .replace(/[ !+=_;:'?.>,<|)(*&^%$#@~`-]+/g, "")
        .toUpperCase();
      const input = courseInput
        .trim()
        .replace(/[ !+=_;:'?.>,<|)(*&^%$#@~`-]+/g, "");
      if (input.includes(text)) {
        return true;
      }
      return false;
    },
    filterSection(courses) {
      return courses.filter(
        (course) =>
          (!this.selectedDepartment ||
            course.department === this.selectedDepartment) &&
          (!this.selectedSubsemester ||
            (this.selectedSubsemester.date_start.getTime() ===
              course.date_start.getTime() &&
              this.selectedSubsemester.date_end.getTime() ===
                course.date_end.getTime()))
      );
    },
  },
  watch: {
    // Remove the textSearch watcher
  },
  computed: {
    ...mapState(["selectedSemester", "subsemesters", "departments"]),
    fullList() {
      return this.$store.getters.courses;
    },
    departmentOptions() {
      return [{ text: "All", value: null }].concat(
        ...this.departments.map(({ department }) => department)
      );
    },

    subsemesterOptions() {
      let options = [{ text: "All", value: null }];
      options.push(
        ...this.subsemesters.map((subsemester) => {
          return { text: subsemester.display_string, value: subsemester };
        })
      );
      // Once we get new data for the <select>, v-model will retain its old value.
      // Need to update this value after receving new data to keep values consistent.
      // eslint-disable-next-line
      this.selectedSubsemester = options[0].value;
      return options;
    },
    // returns exact match if possible.
    // if no exact match exists, returns similar options.
    filterCourses: function () {
      const courses =
        this.courseList !== null
          ? this.courseList
          : this.$store.getters.courses;

      const filtered = this.filterSection(courses);

      //returns exact match, if not found, then department filtered list
      const find = filtered.find(
        (course) =>
          (course.full_title &&
            course.full_title.toUpperCase() ===
              this.textSearch.toUpperCase()) ||
          course.title.toUpperCase() === this.textSearch.toUpperCase()
      );

      const fullListFiltered = this.filterSection(this.fullList);
      const containString = fullListFiltered.filter(
        (course) =>
          this.checkFunction(course.title, this.textSearch) ||
          this.checkFunction(course.department + course.level, this.textSearch)
      );

      if (find) {
        return [find];
      } else {
        return containString;
      }
    },
  },
};
</script>

<style lang="scss">
.course-list {
  display: flex;
  flex-direction: column;
  height: 100%;
  
  .course-search {
    flex: 0 0 auto;
    padding: 1rem;
  }

  #scroll-box {
    flex: 1 1 auto;
    overflow-y: auto;
    min-height: 300px;
  }

  .course-listing {
    background-color: transparent;

    &.bg-light {
      background-color: var(--light) !important;
    }
  }

  // Dark mode styles
  .dark & {
    .course-listing {
      &.bg-light {
        background-color: var(--dark-secondary) !important;
      }
    }
  }
}
</style>
