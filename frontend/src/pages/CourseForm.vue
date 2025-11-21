<template>
	<div class="h-full">
		<div class="grid grid-cols-1 md:grid-cols-[70%,30%] h-full">
			<div>
				<header
					class="sticky top-0 z-10 flex flex-col md:flex-row md:items-center justify-between border-b bg-surface-white px-3 py-2.5 sm:px-5"
				>
					<Breadcrumbs class="h-7" :items="breadcrumbs" />
					<div class="flex items-center mt-3 md:mt-0">
						<Button v-if="courseResource.data?.name" @click="trashCourse()">
							<template #icon>
								<Trash2 class="w-4 h-4 stroke-1.5" />
							</template>
						</Button>
						<Button variant="solid" @click="submitCourse()" class="ml-2">
							<span>
								{{ __("Save") }}
							</span>
						</Button>
					</div>
				</header>
				<div class="mt-5 mb-5">
					<div class="px-5 md:px-10 pb-5 mb-5 space-y-5 border-b">
						<div class="text-lg font-semibold mb-4 text-ink-gray-9">
							{{ __("Details") }}
						</div>
						<div class="grid grid-cols-1 md:grid-cols-2 gap-5">
							<FormControl
								v-model="course.title"
								:label="__('Title')"
								:required="true"
							/>
							<Link
								doctype="LMS Category"
								v-model="course.category"
								:label="__('Category')"
								:onCreate="(value, close) => openSettings('Categories', close)"
								:required="true"
							/>
							<Link
								doctype="Client"
								v-model="course.client"
								:label="__('Client')"
								:required="true"
							/>
							<Link
								doctype="Proces"
								v-model="course.proces"
								:label="__('Process')"
								:required="true"
							/>

							<!-- Set Rules Field - Available for all categories (for examination purposes) -->
							<FormControl
								v-model="course.set_rules"
								:label="__('Set Rules')"
								type="select"
								:options="setRulesOptions"
							/>

							<!-- Conditional fields based on Set Rules -->
							<template v-if="course.set_rules === 'Expiry Period'">
								<FormControl
									v-model="course.expiry_months"
									:label="__('Select Months')"
									type="select"
									:options="[
										{ label: __('3 Months'), value: '3 Months' },
										{ label: __('6 Months'), value: '6 Months' },
										{ label: __('1 Year'), value: '1 Year' },
									]"
								/>
								<div></div>
							</template>

							<template v-if="course.set_rules === 'Prelim Period'">
								<FormControl
									v-model="course.prelim_period"
									:label="__('Prelim Period')"
									type="select"
									:options="[
										{ label: __('15 Days'), value: '15 Days' },
										{ label: __('45 Days'), value: '45 Days' },
										{ label: __('60 Days'), value: '60 Days' },
										{ label: __('90 Days'), value: '90 Days' },
									]"
								/>
								<div></div>
							</template>

							<template v-if="course.set_rules === 'Re-assigned'">
								<FormControl
									v-model="course.assign_date"
									:label="__('Assign Date')"
									type="date"
									:required="true"
									:min="today"
									@change="validateAssignDate"
								/>
								<FormControl
									v-model="course.reassign_after"
									:label="__('Re-assign After')"
									type="select"
									:options="[
										{ label: __('3 Months'), value: '3 Months' },
										{ label: __('6 Months'), value: '6 Months' },
										{ label: __('1 Year'), value: '1 Year' },
									]"
									:required="true"
								/>
							</template>

							<template v-if="course.set_rules === 'Mandatory Course'">
								<FormControl
									v-model="course.assign_date"
									:label="__('Assign Date')"
									type="date"
									:required="true"
									:min="today"
									@change="validateAssignDate"
								/>
								<FormControl
									v-model="course.employee_type"
									:label="__('Employee Type')"
									type="select"
									:options="[
										{ label: __('All Employee'), value: 'All Employee' },
										{
											label: __('Existing Employee'),
											value: 'Existing Employee',
										},
										{ label: __('New Employee'), value: 'New Employee' },
										{
											label: __('Individual Employee'),
											value: 'Individual Employee',
										},
									]"
									:required="true"
								/>
								<Link
									doctype="Department"
									v-model="course.department"
									:label="__('Department')"
									:required="true"
								/>
								<FormControl
									type="checkbox"
									v-model="course.all_locations"
									:label="__('All Locations')"
								/>
								<FormControl
									type="checkbox"
									v-model="course.all_designations"
									:label="__('All Designations')"
								/>
								<MultiSelect
									v-if="!course.all_locations"
									doctype="Locations"
									v-model="locations"
									:label="__('Location')"
									:required="true"
								/>
								<Link
									v-if="!course.all_designations"
									doctype="Designation"
									v-model="course.designation"
									:label="__('Designation')"
									:required="true"
								/>
								<div class="col-span-2">
									<FormControl
										type="checkbox"
										v-model="course.enforce_completion"
										:label="__('Enforce Completion')"
									/>
									<p class="text-xs text-ink-gray-5 mt-1 ml-6">
										{{
											__(
												"Users will receive repeated reminders until they acknowledge and complete this mandatory course"
											)
										}}
									</p>
								</div>
							</template>
						</div>

						<div class="grid grid-cols-1 md:grid-cols-2 gap-5">
							<div>
								<div class="text-xs text-ink-gray-5">
									{{ __("Tags") }}
								</div>
								<FormControl
									v-model="newTag"
									:placeholder="__('Add a keyword and then press enter')"
									:class="['w-full', 'flex-1', 'my-1']"
									@keyup.enter="updateTags()"
									id="tags"
								/>
								<div>
									<div class="flex items-center flex-wrap gap-2">
										<div
											v-if="course.tags"
											v-for="tag in course.tags?.split(', ')"
											class="flex items-center bg-surface-gray-2 text-ink-gray-7 p-2 rounded-md"
										>
											{{ tag }}
											<X
												class="stroke-1.5 w-3 h-3 ml-2 cursor-pointer"
												@click="removeTag(tag)"
											/>
										</div>
									</div>
								</div>
							</div>
						</div>

						<div class="grid grid-cols-1 md:grid-cols-2 gap-5">
							<div class="mb-4">
								<div class="text-xs text-ink-gray-5 mb-2">
									{{ __("Course Image") }}
								</div>
								<FileUploader
									v-if="!course.course_image"
									:fileTypes="['image/*']"
									:validateFile="validateFile"
									@success="(file) => saveImage(file)"
								>
									<template
										v-slot="{ file, progress, uploading, openFileSelector }"
									>
										<div class="flex items-center">
											<div
												class="border rounded-md w-fit py-5 px-20 cursor-pointer"
												@click="openFileSelector"
											>
												<Image class="size-5 stroke-1 text-ink-gray-7" />
											</div>
											<div class="ml-4">
												<Button @click="openFileSelector">
													{{ __("Upload") }}
												</Button>
												<div
													class="mt-1 text-ink-gray-5 text-sm leading-5"
												>
													{{
														__(
															"Appears on the course card in the course list"
														)
													}}
												</div>
											</div>
										</div>
									</template>
								</FileUploader>
								<div v-else class="mb-4">
									<div class="flex items-center">
										<img
											:src="course.course_image.file_url"
											class="border rounded-md w-40"
										/>
										<div class="ml-4">
											<Button @click="removeImage()">
												{{ __("Remove") }}
											</Button>
											<div class="mt-2 text-ink-gray-5 text-sm">
												{{
													__(
														"Appears on the course card in the course list"
													)
												}}
											</div>
										</div>
									</div>
								</div>
							</div>

							<ColorSwatches
								v-model="course.card_gradient"
								:label="__('Color')"
								:description="__('Choose a color for the course card')"
								class="w-full"
							/>
						</div>
					</div>

					<div class="px-5 md:px-10 pb-5 mb-5 space-y-5 border-b">
						<div class="text-lg font-semibold text-ink-gray-9">
							{{ __("Settings") }}
						</div>
						<div class="grid grid-cols-1 md:grid-cols-2 gap-5">
							<div v-if="user.data?.is_moderator" class="flex flex-col space-y-5">
								<FormControl
									type="checkbox"
									v-model="course.published"
									:label="__('Published')"
								/>
								<FormControl
									v-model="course.published_on"
									:label="__('Published On')"
									type="date"
									:min="today"
									@change="validatePublishedDate"
								/>
							</div>
							<div class="flex flex-col space-y-5">
								<FormControl
									type="checkbox"
									v-model="course.upcoming"
									:label="__('Upcoming')"
								/>
								<FormControl
									type="checkbox"
									v-model="course.featured"
									:label="__('Featured')"
								/>
								<FormControl
									type="checkbox"
									v-model="course.disable_self_learning"
									:label="__('Disable Self Enrollment')"
								/>
							</div>
						</div>
					</div>

					<div class="px-5 md:px-10 pb-5 mb-5 space-y-5 border-b">
						<div class="text-lg font-semibold text-ink-gray-9">
							{{ __("About the Course") }}
						</div>
						<FormControl
							v-model="course.short_introduction"
							type="textarea"
							:rows="5"
							:label="__('Short Introduction')"
							:placeholder="
								__(
									'A one line introduction to the course that appears on the course card'
								)
							"
							:required="true"
						/>
						<div class="">
							<div class="mb-1.5 text-sm text-ink-gray-5">
								{{ __("Course Description") }}
								<span class="text-ink-red-3">*</span>
							</div>
							<TextEditor
								:content="course.description"
								@change="(val) => (course.description = val)"
								:editable="true"
								:fixedMenu="true"
								editorClass="prose-sm max-w-none border-b border-x bg-surface-gray-2 rounded-b-md py-1 px-2 min-h-[7rem]"
							/>
						</div>

						<FormControl
							v-model="course.video_link"
							:label="__('Preview Video')"
							:placeholder="
								__(
									'Paste the youtube link of a short video introducing the course'
								)
							"
						/>

						<MultiSelect
							v-model="related_courses"
							doctype="LMS Course"
							:label="__('Related Courses')"
							:filters="{ name: ['!=', courseResource.data?.name] }"
							:onCreate="
								(close) => {
									router.push({
										name: 'CourseForm',
										params: { courseName: 'new' },
									});
								}
							"
						/>
					</div>

					<div class="px-5 md:px-10 pb-5 mb-5 space-y-5 border-b">
						<div class="text-lg font-semibold text-ink-gray-9">
							{{ __("Final Exam") }}
						</div>
						<Link
							doctype="LMS Quiz"
							v-model="course.final_exam"
							:label="__('Select a Final Exam')"
						/>
					</div>

					<div class="px-5 md:px-10 pb-5 space-y-5 border-b">
						<div class="text-lg font-semibold mt-5">
							{{ __("Certification") }}
						</div>
						<div class="grid grid-cols-1 md:grid-cols-3 gap-5">
							<FormControl
								type="checkbox"
								v-model="course.enable_certification"
								:label="__('Completion Certificate')"
							/>
						</div>
						<div
							v-if="course.enable_certification"
							class="grid grid-cols-1 md:grid-cols-2 gap-5"
						>
							<div>
								<Link
									doctype="Print Format"
									v-model="course.custom_certificate_template"
									:label="__('Certificate Template')"
									:filters="{ doc_type: 'LMS Certificate' }"
								/>
								<!-- Preview Button -->
								<Button
									v-if="course.custom_certificate_template"
									@click="previewCertificate()"
									class="mt-2"
									variant="subtle"
								>
									{{ __("Preview Certificate") }}
								</Button>
							</div>
						</div>
						<div
							v-if="course.enable_certification"
							class="grid grid-cols-1 md:grid-cols-2 gap-5"
						>
							<div>
								<Link
									doctype="Employee"
									v-model="course.custom_signature_1_from"
									:label="__('Signature 1 From')"
									:filters="{ status: 'Active' }"
								/>
							</div>
							<div v-if="course.custom_signature_1_from" class="mb-4">
								<div class="text-xs text-ink-gray-5 mb-2">
									{{ __("Signature 1") }}
								</div>
								<FileUploader
									v-if="!course.custom_signature_1"
									:fileTypes="['image/*']"
									:validateFile="validateFile"
									@success="(file) => saveSignature1(file)"
								>
									<template
										v-slot="{ file, progress, uploading, openFileSelector }"
									>
										<div class="flex items-center">
											<div
												class="border rounded-md w-fit py-5 px-20 cursor-pointer"
												@click="openFileSelector"
											>
												<Image class="size-5 stroke-1 text-ink-gray-7" />
											</div>
											<div class="ml-4">
												<Button @click="openFileSelector">
													{{ __("Upload") }}
												</Button>
											</div>
										</div>
									</template>
								</FileUploader>
								<div v-else class="mb-4">
									<div class="flex items-center">
										<img
											:src="course.custom_signature_1.file_url"
											class="border rounded-md w-40"
										/>
										<div class="ml-4">
											<Button @click="removeSignature1()">
												{{ __("Remove") }}
											</Button>
										</div>
									</div>
								</div>
							</div>
						</div>
						<div
							v-if="course.enable_certification"
							class="grid grid-cols-1 md:grid-cols-2 gap-5"
						>
							<div>
								<Link
									doctype="Employee"
									v-model="course.custom_signature_2_from"
									:label="__('Signature 2 From')"
									:filters="{ status: 'Active' }"
								/>
							</div>
							<div v-if="course.custom_signature_2_from" class="mb-4">
								<div class="text-xs text-ink-gray-5 mb-2">
									{{ __("Signature 2") }}
								</div>
								<FileUploader
									v-if="!course.custom_signature_2"
									:fileTypes="['image/*']"
									:validateFile="validateFile"
									@success="(file) => saveSignature2(file)"
								>
									<template
										v-slot="{ file, progress, uploading, openFileSelector }"
									>
										<div class="flex items-center">
											<div
												class="border rounded-md w-fit py-5 px-20 cursor-pointer"
												@click="openFileSelector"
											>
												<Image class="size-5 stroke-1 text-ink-gray-7" />
											</div>
											<div class="ml-4">
												<Button @click="openFileSelector">
													{{ __("Upload") }}
												</Button>
											</div>
										</div>
									</template>
								</FileUploader>
								<div v-else class="mb-4">
									<div class="flex items-center">
										<img
											:src="course.custom_signature_2.file_url"
											class="border rounded-md w-40"
										/>
										<div class="ml-4">
											<Button @click="removeSignature2()">
												{{ __("Remove") }}
											</Button>
										</div>
									</div>
								</div>
							</div>
						</div>
						<div class="grid grid-cols-1 md:grid-cols-2 gap-5">
							<div class="space-y-5">
								<FormControl
									v-if="course.paid_course || course.paid_certificate"
									v-model="course.course_price"
									:label="__('Amount')"
								/>
								<Link
									v-if="course.paid_certificate"
									doctype="Course Evaluator"
									v-model="course.evaluator"
									:label="__('Evaluator')"
									:onCreate="(value, close) => openSettings('Evaluators', close)"
								/>
							</div>
							<Link
								v-if="course.paid_course || course.paid_certificate"
								doctype="Currency"
								v-model="course.currency"
								:filters="{ enabled: 1 }"
								:label="__('Currency')"
							/>
						</div>
					</div>

					<div class="px-5 md:px-10 pb-5 space-y-5">
						<div class="text-lg font-semibold mt-5 text-ink-gray-9">
							{{ __("Meta Tags") }}
						</div>
						<div class="space-y-5">
							<FormControl
								v-model="meta.description"
								:label="__('Meta Description')"
								type="textarea"
								:rows="7"
							/>
							<FormControl
								v-model="meta.keywords"
								:label="__('Meta Keywords')"
								type="textarea"
								:rows="7"
								:placeholder="__('Comma separated keywords for SEO')"
							/>
						</div>
					</div>
				</div>
			</div>
			<div class="border-l">
				<CourseOutline
					v-if="courseResource.data"
					:courseName="courseResource.data.name"
					:title="__('Course Outline')"
					:allowEdit="true"
				/>
			</div>
		</div>
	</div>
</template>

<script setup>
import {
	Breadcrumbs,
	call,
	TextEditor,
	Button,
	createResource,
	FormControl,
	FileUploader,
	usePageMeta,
	toast,
} from "frappe-ui";
import {
	inject,
	onMounted,
	onBeforeUnmount,
	computed,
	ref,
	reactive,
	watch,
	getCurrentInstance,
} from "vue";
import { Image, Trash2, X } from "lucide-vue-next";
import { useRouter, useRoute } from "vue-router";
import { capture, startRecording, stopRecording } from "@/telemetry";
import { useOnboarding } from "frappe-ui/frappe";
import { sessionStore } from "../stores/session";
import { openSettings, getMetaInfo, updateMetaInfo, validateFile } from "@/utils";
import Link from "@/components/Controls/Link.vue";
import CourseOutline from "@/components/CourseOutline.vue";
import MultiSelect from "@/components/Controls/MultiSelect.vue";
import ColorSwatches from "@/components/Controls/ColorSwatches.vue";

const user = inject("$user");
const newTag = ref("");
const { brand } = sessionStore();
const router = useRouter();
const route = useRoute();
const locations = ref([]);
const related_courses = ref([]);
const app = getCurrentInstance();
const { updateOnboardingStep } = useOnboarding("learning");
const { $dialog } = app.appContext.config.globalProperties;

const props = defineProps({
	courseName: {
		type: String,
	},
});

const course = reactive({
	title: "",
	short_introduction: "",
	description: "",
	video_link: "",
	course_image: null,
	custom_signature_1: null,
	custom_signature_2: null,
	card_gradient: "",
	tags: "",
	category: "",
	client: "",
	proces: "",
	set_rules: "",
	expiry_months: "",
	prelim_period: "",
	reassign_after: "",
	assign_date: new Date().toISOString().split("T")[0],
	employee_type: "",
	location: "",
	department: "",
	designation: "",
	enforce_completion: false,
	all_locations: false,
	all_designations: false,
	instructors: "",
	published: false,
	published_on: new Date().toISOString().split("T")[0],
	featured: false,
	upcoming: false,
	disable_self_learning: false,
	enable_certification: false,
	custom_certificate_template: "",
	custom_signature_1_from: "",
	custom_signature_2_from: "",
	paid_course: false,
	paid_certificate: false,
	course_price: "",
	currency: "",
	evaluator: "",
	timezone: "",
	final_exam: "",
});

const meta = reactive({
	description: "",
	keywords: "",
});

onMounted(() => {
	if (!user.data?.is_moderator && !user.data?.is_instructor) {
		router.push({ name: "Courses" });
	}

	if (props.courseName !== "new") {
		fetchCourseInfo();
	} else {
		if (route.query.category) {
			course.category = route.query.category;
		}
		capture("course_form_opened");
		startRecording();
	}
	window.addEventListener("keydown", keyboardShortcut);
});

const fetchCourseInfo = () => {
	courseResource.reload();
	getMetaInfo("courses", props.courseName, meta);
};

const keyboardShortcut = (e) => {
	if (e.key === "s" && (e.ctrlKey || e.metaKey) && !e.target.classList.contains("ProseMirror")) {
		submitCourse();
		e.preventDefault();
	}
};

onBeforeUnmount(() => {
	window.removeEventListener("keydown", keyboardShortcut);
	stopRecording();
});

const courseCreationResource = createResource({
	url: "frappe.client.insert",
	makeParams(values) {
		// Create a clean copy of course data without the file objects
		const cleanCourse = { ...values };

		// Remove file objects from the clean copy
		delete cleanCourse.course_image;
		delete cleanCourse.custom_signature_1;
		delete cleanCourse.custom_signature_2;

		return {
			doc: {
				doctype: "LMS Course",
				image: course.course_image?.file_url || "",
				custom_signature_1: course.custom_signature_1?.file_url || "",
				custom_signature_2: course.custom_signature_2?.file_url || "",
				locations: locations.value.map((location) => ({ location })),
				related_courses: related_courses.value.map((course) => ({ course })),
				all_locations: course.all_locations ? 1 : 0,
				all_designations: course.all_designations ? 1 : 0,
				...cleanCourse,
			},
		};
	},
});

const courseEditResource = createResource({
	url: "frappe.client.set_value",
	auto: false,
	makeParams(values) {
		// Create a clean copy of course data without the file objects
		const cleanCourse = { ...course };

		// Replace file objects with their URLs
		delete cleanCourse.course_image;
		delete cleanCourse.custom_signature_1;
		delete cleanCourse.custom_signature_2;

		return {
			doctype: "LMS Course",
			name: values.course,
			fieldname: {
				image: course.course_image?.file_url || "",
				custom_signature_1: course.custom_signature_1?.file_url || "",
				custom_signature_2: course.custom_signature_2?.file_url || "",
				custom_signature_1_from: course.custom_signature_1_from || "",
				custom_signature_2_from: course.custom_signature_2_from || "",
				locations: locations.value.map((location) => ({ location })),
				related_courses: related_courses.value.map((course) => ({ course })),
				all_locations: course.all_locations ? 1 : 0,
				all_designations: course.all_designations ? 1 : 0,
				...cleanCourse,
			},
		};
	},
});

const courseResource = createResource({
	url: "frappe.client.get",
	makeParams(values) {
		return {
			doctype: "LMS Course",
			name: props.courseName,
		};
	},
	auto: false,
	onSuccess(data) {
		Object.keys(data).forEach((key) => {
			if (key === "all_locations") course.all_locations = data[key] ? true : false;
			if (key === "all_designations") course.all_designations = data[key] ? true : false;
			if (key == "locations") {
				locations.value = [];
				data.locations.forEach((location) => {
					locations.value.push(location.location);
				});
			} else if (key == "related_courses") {
				related_courses.value = [];
				data.related_courses.forEach((course) => {
					related_courses.value.push(course.course);
				});
			} else if (Object.hasOwn(course, key)) course[key] = data[key];
		});

		let checkboxes = [
			"published",
			"upcoming",
			"disable_self_learning",
			"paid_course",
			"featured",
			"enable_certification",
			"paid_certificate",
			"enforce_completion",
		];
		for (let idx in checkboxes) {
			let key = checkboxes[idx];
			course[key] = course[key] ? true : false;
		}

		// Load all images
		if (data.image) {
			imageResource.reload({ image: data.image });
		}
		if (data.custom_signature_1) {
			signature1Resource.reload({ image: data.custom_signature_1 });
		}
		if (data.custom_signature_2) {
			signature2Resource.reload({ image: data.custom_signature_2 });
		}

		check_permission();
	},
});

const imageResource = createResource({
	url: "lms.lms.api.get_file_info",
	makeParams(values) {
		return {
			file_url: values.image,
		};
	},
	auto: false,
	onSuccess(data) {
		course.course_image = data;
	},
});

const signature1Resource = createResource({
	url: "lms.lms.api.get_file_info",
	makeParams(values) {
		return {
			file_url: values.image,
		};
	},
	auto: false,
	onSuccess(data) {
		course.custom_signature_1 = data;
	},
});

const signature2Resource = createResource({
	url: "lms.lms.api.get_file_info",
	makeParams(values) {
		return {
			file_url: values.image,
		};
	},
	auto: false,
	onSuccess(data) {
		course.custom_signature_2 = data;
	},
});

const submitCourse = () => {
	if (courseResource.data) {
		editCourse();
	} else {
		createCourse();
	}
};

const createCourse = () => {
	courseCreationResource.submit(course, {
		onSuccess(data) {
			updateMetaInfo("courses", data.name, meta);
			if (user.data?.is_system_manager) {
				updateOnboardingStep("create_first_course", true, false, () => {
					localStorage.setItem("firstCourse", data.name);
				});
			}

			capture("course_created");
			toast.success(__("Course created successfully"));
			router.push({
				name: "CourseForm",
				params: { courseName: data.name },
			});
		},
		onError(err) {
			toast.error(err.messages?.[0] || err);
		},
	});
};

const editCourse = () => {
	courseEditResource.submit(
		{
			course: courseResource.data.name,
		},
		{
			onSuccess() {
				updateMetaInfo("courses", props.courseName, meta);
				toast.success(__("Course updated successfully"));
			},
			onError(err) {
				toast.error(err.messages?.[0] || err);
			},
		}
	);
};

const deleteCourse = createResource({
	url: "lms.lms.api.delete_course",
	makeParams(values) {
		return {
			course: props.courseName,
		};
	},
	onSuccess() {
		toast.success(__("Course deleted successfully"));
		router.push({ name: "Courses" });
	},
});

const trashCourse = () => {
	$dialog({
		title: __("Delete Course"),
		message: __(
			"Deleting the course will also delete all its chapters and lessons. Are you sure you want to delete this course?"
		),
		actions: [
			{
				label: __("Delete"),
				theme: "red",
				variant: "solid",
				onClick(close) {
					deleteCourse.submit();
					close();
				},
			},
		],
	});
};

watch(
	() => props.courseName !== "new",
	(newVal) => {
		if (newVal) {
			fetchCourseInfo();
		}
	}
);

const updateTags = () => {
	if (newTag.value) {
		course.tags = course.tags ? `${course.tags}, ${newTag.value}` : newTag.value;
		newTag.value = "";
	}
};

const removeTag = (tag) => {
	course.tags = course.tags
		?.split(", ")
		.filter((t) => t !== tag)
		.join(", ");
	newTag.value = "";
};

const saveImage = (file) => {
	course.course_image = file;
};

const saveSignature1 = (file) => {
	course.custom_signature_1 = file;
};

const saveSignature2 = (file) => {
	course.custom_signature_2 = file;
};

const removeImage = () => {
	course.course_image = null;
};

const removeSignature1 = () => {
	course.custom_signature_1 = null;
};

const removeSignature2 = () => {
	course.custom_signature_2 = null;
};

const previewCertificate = () => {
	if (!course.custom_certificate_template) {
		toast.error(__("Please select a certificate template first"));
		return;
	}

	const url = `/api/method/fusion_payroll.overrides.lms_course.preview_certificate_template?format_name=${course.custom_certificate_template}`;
	window.open(url, "_blank");
};

const check_permission = () => {
	if (user.data?.is_moderator) return;

	if (course.instructors !== user.data?.name) {
		router.push({ name: "Courses" });
	}
};

const breadcrumbs = computed(() => {
	let crumbs = [];

	// If we came from a category page, include it in breadcrumbs
	if (route.query.category && route.query.fromCategory) {
		crumbs.push({
			label: "Categories",
			route: { name: "Categories" },
		});
		crumbs.push({
			label: route.query.category,
			route: {
				name: "CategoryDetail",
				params: { categoryName: route.query.category },
			},
		});
	} else {
		crumbs.push({
			label: "Courses",
			route: { name: "Courses" },
		});
	}

	if (courseResource.data) {
		crumbs.push({
			label: course.title,
			route: { name: "CourseDetail", params: { courseName: props.courseName } },
		});
	}
	crumbs.push({
		label: props.courseName == "new" ? "New Course" : "Edit Course",
		route: { name: "CourseForm", params: { courseName: props.courseName } },
	});
	return crumbs;
});

usePageMeta(() => {
	return {
		title: courseResource.data?.title || __("New Course"),
		icon: brand.favicon,
	};
});

const today = computed(() => {
	return new Date().toISOString().split("T")[0];
});

const validateAssignDate = () => {
	if (course.assign_date && course.assign_date < today.value) {
		toast.error(__("Assign date cannot be in the past"));
		course.assign_date = today.value;
	}
};

const validatePublishedDate = () => {
	if (course.published_on && course.published_on < today.value) {
		toast.error(__("Published date cannot be in the past"));
		course.published_on = today.value;
	}
};

// Set rules options - available for all categories
const setRulesOptions = computed(() => {
	return [
		{ label: __("No Expiry Period"), value: "No Expiry Period" },
		{ label: __("Expiry Period"), value: "Expiry Period" },
		{ label: __("Before Confirmation"), value: "Before Confirmation" },
		{ label: __("Prelim Period"), value: "Prelim Period" },
		{ label: __("Re-assigned"), value: "Re-assigned" },
		{ label: __("Mandatory Course"), value: "Mandatory Course" },
	];
});
</script>
