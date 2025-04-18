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
      <CourseListing 
        :course="course" 
        :showAddButton="false"
        defaultAction="toggleCollapse"
        @addCourseSection="$emit('addCourseSection', $event[0], $event[1])"
        @removeCourseSection="$emit('removeCourseSection', $event)"
        @removeCourse="$emit('removeCourse', $event)"
      />
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

<style scoped>
.no-courses {
  text-align: center;
  padding: 20px;
  color: #666;
}
</style>
