<script setup>
import { useForm } from '@inertiajs/vue3'
import { FwbButton, FwbInput } from 'flowbite-vue'
import { computed, ref } from 'vue'
import InputLabel from '@/Components/InputLabel.vue'
import InputError from '@/Components/InputError.vue'
import MemberQualificationsList from '@/Components/MemberQualificationsList.vue'
import DeleteConfirmationModal from '@/Components/DeleteConfirmationModal.vue'
import DialogModal from './DialogModal.vue'
import AddButton from '@/Components/AddButton.vue'
import DeleteButton from '@/Components/DeleteButton.vue'
import UpdateButton from '@/Components/UpdateButton.vue'
import CancelButton from '@/Components/CancelButton.vue'

const props = defineProps({
  member_id: Number,
  list: {
    type: Object,
    default: [],
  },
  countryList: {
    type: Object,
    default: [],
  },
  editable: {
    type: Boolean,
    default: true,
  },
})

const form = useForm({
  country_iso2: null,
  qualification: null,
  year_attained: null,
  institution: null,
})

const listData = props.list
const itemId = ref(-1)

const popularCountries = ['AU', 'FJ', 'NZ', 'WS', 'VU', 'US']
const showFormModal = ref(false)
const showConfirmationModal = ref(false)

const canAdd = computed(() => {
  return itemId.value < 0
})

const canEdit = computed(() => {
  return !canAdd.value
})

const otherCountries = computed(() => {
  let countries = [],
    keys = Object.keys(props.countryList)

  keys.forEach(c => {
    if (!popularCountries.includes(c)) {
      countries.push(c)
    }
  })
  return countries
})

function closeModal() {
  showFormModal.value = false
}
function closeModalAndResetForm() {
  closeModal()
  itemId.value = -1
  form.reset()
  form.clearErrors()
}
function showModal() {
  showFormModal.value = true
}
function edit(id) {
  itemId.value = id
  let item = listData.find(i => i.id === id)

  form.qualification = item.qualification
  form.year_attained = item.year_attained
  form.institution = item.institution
  form.country_iso2 = item.country_iso2
  showModal()
}
function submit() {
  form.post(route('members.qualifications.store', props.member_id), {
    onSuccess(res) {
      let formCopy = Object.assign({}, form)
      formCopy.id = res.props.flash.data.id
      listData.push(formCopy)

      // reset form
      closeModalAndResetForm()
    },
  })
}
function update() {
  form.put(route('members.qualifications.update', { member: props.member_id, qualification: itemId.value }), {
    preserveScroll: true,
    resetOnSuccess: false,
    onSuccess() {
      let item = listData.find(i => i.id === itemId.value)
      item.qualification = form.qualification
      item.year_attained = form.year_attained
      item.institution = form.institution
      item.country_iso2 = form.country_iso2

      // reset form
      closeModalAndResetForm()
    },
  })
}
function deleteItem() {
  form.delete(route('members.qualifications.destroy', { member: props.member_id, qualification: itemId.value }), {
    preserveScroll: true,
    resetOnSuccess: false,
    onSuccess() {
      for (var i = 0; i < listData.length; i++) {
        if (listData[i].id === itemId.value) {
          listData.splice(i, 1)
        }
      }

      // reset form
      closeModalAndResetForm()
      showConfirmationModal.value = false
    },
  })
}
</script>
<template>
  <div>
    <h5 class="mb-3">Academic Qualifications</h5>

    <fwb-Button v-show="props.editable" class="p-3 mb-3" color="alternative" @click.prevent="showModal">Add Qualification</fwb-Button>

    <!-- Member qualifications list -->
    <MemberQualificationsList :list="listData" @edit-item="edit" />
  </div>

  <!-- Modal -->
  <DialogModal :show="showFormModal" @close="closeModalAndResetForm">
    <template #title>
      <div class="flex items-center text-lg">
        <span v-if="canAdd"> Add Qualification </span>
        <span v-else> Edit Qualification </span>
      </div>
    </template>
    <template #content>
      <fwb-Input v-model="form.qualification" placeholder="enter your qualification" label="Qualification" class="mb-2" />
      <InputError class="mt-2" :message="form.errors.qualification" />

      <fwb-Input v-model="form.year_attained" name="year_attained" type="number" placeholder="enter year attained" label="Year attained" class="mb-2" />
      <InputError class="mt-2" :message="form.errors.year_attained" />

      <fwb-Input v-model="form.institution" placeholder="enter your institution" label="Institution" class="mb-2" />
      <InputError class="mt-2" :message="form.errors.institution" />

      <InputLabel for="countries" value="Country" class="mb-2" />
      <select id="countries" v-model="form.country_iso2" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500 mb-3">
        <option selected>Choose a country</option>
        <option disabled>----------</option>
        <option v-for="countryCode in popularCountries" :value="countryCode">{{ props.countryList[countryCode] }}</option>
        <option disabled>--Others--</option>
        <option v-for="countryCode in otherCountries" :value="countryCode">{{ props.countryList[countryCode] }}</option>
      </select>
      <InputError class="mt-2" :message="form.errors.country_iso2" />
    </template>
    <template #footer>
      <CancelButton @click="closeModalAndResetForm" />

      <div>
        <AddButton v-if="canAdd" :disabled="form.processing" @click="submit" />
        <DeleteButton v-if="canEdit" @click="showConfirmationModal = true" />
        <UpdateButton v-if="canEdit" @click="update" />
      </div>
    </template>
  </DialogModal>
  <DeleteConfirmationModal :show="showConfirmationModal" @delete="deleteItem" @close="showConfirmationModal = false" />
</template>
