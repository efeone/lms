<template>
	<div v-if="course.data">
		<header
			class="sticky top-0 z-10 flex items-center justify-between border-b bg-surface-white px-3 py-2.5 sm:px-5"
		>
			<Breadcrumbs class="h-7" :items="breadcrumbs" />
		</header>
		<div class="m-5">
			<div class="flex justify-between w-full space-x-5">
				<div class="md:w-2/3">
					<div class="text-3xl font-semibold text-ink-gray-9">
						{{ course.data.title }}
					</div>
					<div class="my-3 leading-6 text-ink-gray-7">
						{{ course.data.short_introduction }}
					</div>
					<div class="flex items-center">
						<Tooltip
							v-if="parseInt(course.data.rating) > 0"
							:text="__('Average Rating')"
							class="flex items-center"
						>
							<Star class="size-4 text-transparent fill-yellow-500" />
							<span class="ml-1 text-ink-gray-7">
								{{ course.data.rating }}
							</span>
						</Tooltip>
						<span v-if="parseInt(course.data.rating) > 0" class="mx-3"
							>&middot;</span
						>
						<Tooltip
							v-if="course.data.enrollment_count"
							:text="__('Enrolled Students')"
							class="flex items-center"
						>
							<Users class="h-4 w-4 text-ink-gray-7" />
							<span class="ml-1">
								{{ course.data.enrollment_count_formatted }}
							</span>
						</Tooltip>
						<span v-if="course.data.enrollment_count" class="mx-3"
							>&middot;</span
						>
						<div class="flex items-center">
							<span
								class="h-6 mr-1"
								:class="{
									'avatar-group overlap': course.data.instructors.length > 1,
								}"
							>
								<UserAvatar
									v-for="instructor in course.data.instructors"
									:user="instructor"
								/>
							</span>
							<CourseInstructors :instructors="course.data.instructors" />
						</div>
					</div>
					<div v-if="course.data.tags" class="flex my-4 w-fit">
						<Badge
							theme="gray"
							size="lg"
							class="mr-2 text-ink-gray-9"
							v-for="tag in course.data.tags.split(', ')"
						>
							{{ tag }}
						</Badge>
					</div>
					<div class="md:hidden my-4">
						<CourseCardOverlay
							:course="course"
							:isLMSAdmin="isLMSAdmin"
							@openBulkEnroll="showBulkEnrollDialog = true"
						/>
					</div>
					<div
						v-html="course.data.description"
						class="ProseMirror prose prose-table:table-fixed prose-td:p-2 prose-th:p-2 prose-td:border prose-th:border prose-td:border-outline-gray-2 prose-th:border-outline-gray-2 prose-td:relative prose-th:relative prose-th:bg-surface-gray-2 prose-sm max-w-none !whitespace-normal mt-10"
					></div>
					<div class="mt-10">
						<CourseOutline
							:title="__('Course Outline')"
							:courseName="course.data.name"
							:showOutline="true"
							:getProgress="course.data.membership ? true : false"
						/>
					</div>

					<!-- Enhanced Final Exam Section -->
					<div v-if="course.data.final_exam" class="mt-10">
						<div class="text-xl font-semibold text-ink-gray-9 mb-4">
							{{ __('Final Exam') }}
						</div>
						<div class="p-5 border rounded-lg bg-surface-gray-1">
							<div class="flex items-start justify-between">
								<div class="flex-1">
									<div class="text-lg font-medium text-ink-gray-9 mb-2">
										{{ course.data.final_exam }}
									</div>

									<!-- Show exam status if user has attempts -->
									<div v-if="examStatus.data" class="space-y-3">
										<!-- Last attempt info -->
										<div
											v-if="examStatus.data.last_attempt"
											class="flex items-center gap-4 text-sm"
										>
											<div class="flex items-center gap-2">
												<span class="text-ink-gray-7">{{
													__('Last Score:')
												}}</span>
												<Badge
													:theme="examStatus.data.passed ? 'green' : 'orange'"
													size="md"
												>
													{{ examStatus.data.last_attempt.score }}%
												</Badge>
											</div>
											<div
												v-if="examStatus.data.last_attempt.submission_time"
												class="text-ink-gray-6"
											>
												{{
													formatDate(
														examStatus.data.last_attempt.submission_time,
													)
												}}
											</div>
										</div>

										<!-- Pass/Fail status -->
										<div
											v-if="examStatus.data.passed"
											class="flex items-center gap-2 text-green-600"
										>
											<CheckCircle class="w-5 h-5" />
											<span class="font-medium">{{ __('Exam Passed!') }}</span>
										</div>
										<div
											v-else-if="examStatus.data.last_attempt"
											class="flex items-center gap-2 text-orange-600"
										>
											<AlertCircle class="w-5 h-5" />
											<span>{{
												__('Keep trying! You can retake the exam.')
											}}</span>
										</div>

										<!-- Attempts count -->
										<div class="text-sm text-ink-gray-6">
											{{ __('Attempts:') }}
											{{ examStatus.data.attempt_count || 0 }}
										</div>
									</div>

									<!-- First time taking exam message -->
									<div v-else class="text-sm text-ink-gray-6">
										{{ __('Take the final exam to test your knowledge') }}
									</div>
								</div>

								<!-- Action button -->
								<Button
									variant="solid"
									:theme="examStatus.data?.passed ? 'gray' : 'blue'"
									@click="goToExam"
									:loading="examStatus.loading"
								>
									<template v-if="examStatus.data?.passed">
										{{ __('Retake Exam') }}
									</template>
									<template v-else-if="examStatus.data?.last_attempt">
										{{ __('Try Again') }}
									</template>
									<template v-else>
										{{ __('Start Exam') }}
									</template>
								</Button>
							</div>
						</div>
					</div>

					<CourseReviews
						:courseName="course.data.name"
						:avg_rating="course.data.rating"
						:membership="course.data.membership"
					/>
				</div>
				<div class="hidden md:block">
					<CourseCardOverlay
						:course="course"
						:isLMSAdmin="isLMSAdmin"
						@openBulkEnroll="showBulkEnrollDialog = true"
					/>
				</div>
			</div>
			<RelatedCourses :courseName="course.data.name" />
		</div>

		<!-- Bulk Enroll Dialog -->
		<Dialog
			v-model="showBulkEnrollDialog"
			:options="{
				title: __('Enroll Students'),
				size: 'xl',
			}"
		>
			<template #body-content>
				<div class="space-y-4">
					<div>
						<label class="block text-sm font-medium text-ink-gray-9 mb-2">
							{{ __('Select Students') }}
						</label>
						<Autocomplete
							:options="studentOptions"
							v-model="selectedStudents"
							:multiple="true"
							:placeholder="__('Search and select students...')"
						/>
					</div>
					<div
						v-if="selectedStudents.length > 0"
						class="text-sm text-ink-gray-7"
					>
						{{ selectedStudents.length }} {{ __('student(s) selected') }}
					</div>
					<div v-if="enrollmentError" class="text-sm text-red-600">
						{{ enrollmentError }}
					</div>
					<div v-if="enrollmentSuccess" class="text-sm text-green-600">
						{{ enrollmentSuccess }}
					</div>
				</div>
			</template>
			<template #actions>
				<Button
					variant="solid"
					@click="bulkEnrollStudents"
					:loading="isEnrolling"
					:disabled="selectedStudents.length === 0"
				>
					{{ __('Enroll Students') }}
				</Button>
			</template>
		</Dialog>
	</div>
</template>

<script setup>
import {
	createResource,
	Breadcrumbs,
	Badge,
	Tooltip,
	usePageMeta,
	Button,
	Dialog,
	Autocomplete,
	call,
} from 'frappe-ui'
import { computed, inject, watch, ref } from 'vue'
import { Users, Star, CheckCircle, AlertCircle } from 'lucide-vue-next'
import { sessionStore } from '@/stores/session'
import { useRouter, useRoute } from 'vue-router'
import CourseCardOverlay from '@/components/CourseCardOverlay.vue'
import CourseOutline from '@/components/CourseOutline.vue'
import CourseReviews from '@/components/CourseReviews.vue'
import UserAvatar from '@/components/UserAvatar.vue'
import CourseInstructors from '@/components/CourseInstructors.vue'
import RelatedCourses from '@/components/RelatedCourses.vue'

const { brand } = sessionStore()
const router = useRouter()
const route = useRoute()
const user = inject('$user')

// Bulk enrollment state
const showBulkEnrollDialog = ref(false)
const selectedStudents = ref([])
const isEnrolling = ref(false)
const enrollmentError = ref('')
const enrollmentSuccess = ref('')

const props = defineProps({
	courseName: {
		type: String,
		required: true,
	},
})

const course = createResource({
	url: 'lms.lms.utils.get_course_details',
	cache: ['course', props.courseName],
	makeParams() {
		return {
			course: props.courseName,
		}
	},
	auto: true,
})

// Resource to get exam status
const examStatus = createResource({
	url: 'fusion_payroll.level_up.api.api.get_exam_status',
	makeParams() {
		return {
			quiz: course.data?.final_exam,
		}
	},
})

// Watch for course data and load exam status
watch(
	() => course.data?.final_exam,
	(newVal) => {
		if (newVal && course.data?.membership) {
			examStatus.reload()
		}
	},
)

// Check if user has LMS Admin role
const isLMSAdmin = computed(() => {
	return user.data?.roles?.includes('LMS Admin') || false
})

// Resource to fetch students
const students = createResource({
	url: 'frappe.client.get_list',
	makeParams() {
		return {
			doctype: 'User',
			fields: ['name', 'full_name', 'email'],
			filters: {
				enabled: 1,
			},
			limit_page_length: 0,
		}
	},
	auto: isLMSAdmin.value,
})

const studentOptions = computed(() => {
	if (!students.data) return []
	return students.data.map((student) => ({
		label: `${student.full_name || student.name} (${student.email})`,
		value: student.name,
	}))
})

const bulkEnrollStudents = async () => {
	if (selectedStudents.value.length === 0) return

	isEnrolling.value = true
	enrollmentError.value = ''
	enrollmentSuccess.value = ''

	try {
		const studentEmails = selectedStudents.value.map((student) =>
			typeof student === 'object' ? student.value : student,
		)

		await call('fusion_payroll.level_up.api.api.bulk_enroll_students', {
			course: props.courseName,
			students: studentEmails,
		})

		enrollmentSuccess.value = `Successfully enrolled ${selectedStudents.value.length} student(s)`
		selectedStudents.value = []
		course.reload()

		setTimeout(() => {
			showBulkEnrollDialog.value = false
			enrollmentSuccess.value = ''
		}, 2000)
	} catch (error) {
		enrollmentError.value = error.message || 'Failed to enroll students'
	} finally {
		isEnrolling.value = false
	}
}

const goToExam = () => {
	router.push({
		name: 'QuizPage',
		params: {
			quizID: course.data.final_exam,
		},
		query: {
			returnTo: route.fullPath, // Pass current route for back navigation
		},
	})
}

const formatDate = (dateString) => {
	if (!dateString) return ''
	const date = new Date(dateString)
	return date.toLocaleDateString('en-US', {
		month: 'short',
		day: 'numeric',
		year: 'numeric',
		hour: '2-digit',
		minute: '2-digit',
	})
}

watch(
	() => props.courseName,
	() => {
		course.reload()
	},
)

watch(course, () => {
	if (
		!isInstructor() &&
		!user.data?.is_moderator &&
		!course.data?.published &&
		!course.data?.upcoming
	) {
		router.push({
			name: 'Courses',
		})
	}
})

const isInstructor = () => {
	let user_is_instructor = false
	course.data?.instructors.forEach((instructor) => {
		if (!user_is_instructor && instructor.name == user.data?.name) {
			user_is_instructor = true
		}
	})
	return user_is_instructor
}

const breadcrumbs = computed(() => {
	let items = [
		{
			label: 'Categories',
			route: { name: 'Categories' },
		},
		{
			label: 'Courses',
			route: { name: 'Courses' },
		},
	]
	items.push({
		label: course?.data?.title,
		route: { name: 'CourseDetail', params: { courseName: course?.data?.name } },
	})
	return items
})

usePageMeta(() => {
	return {
		title: course?.data?.title,
		icon: brand.favicon,
	}
})
</script>

<style>
.avatar-group {
	display: inline-flex;
	align-items: center;
}

.avatar-group .avatar {
	transition: margin 0.1s ease-in-out;
}
</style>
