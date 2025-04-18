<template>
  <b-list-group id="selected-course-list" flush data-cy="selected-courses">
    <div v-if="Object.keys(courses).length == 0" class="no-courses">
      Oops! It looks like you haven't selected anything!
      <br />
      Please select some courses from the "Course Search" tab!
    </div>
    <b-list-group-item
      class="selected"
      v-for="course of courses"
      :key="course.id"
    >
      <CourseListing :course="course" v-on="$listeners" />
    </b-list-group-item>
  </b-list-group>
</template>

<script>
import "@/typedef";

import CourseListingComponent from "@/components/CourseListing";

export default {
  name: "SelectedCourses",
  components: {
    CourseListing: CourseListingComponent,
  },
  props: {
    courses: Object,
  },
};
</script>

<style lang="scss">
#selected-course-list {
  height: 100%;
  min-height: 300px;
  overflow-y: auto;
  border: 1px solid #dee2e6;
  border-radius: 0.25rem;
  background-color: transparent;
}

.no-courses {
  padding: 2rem;
  text-align: center;
  color: #6c757d;
}

// Dark mode styles
.dark {
  #selected-course-list {
    border-color: var(--dark-border-primary);
  }

  .no-courses {
    color: var(--dark-text-secondary);
  }
}
</style>
