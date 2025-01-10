<template>
  <div v-if="!movieResources.loading && movieResources.doc">
    <h1 class="text-grey-900 font-bold text-[32px]">{{ movieDoc.title }}</h1>
    <div class="mt-11 flex flex-row items-center justify-between">
      <div class="flex flex-col space-y-3">
        <h2 class="text-grey-700 text-base font-bold uppercase">Director</h2>
        <h2 class="text-grey-600 text-xl font-semibold">
          {{ movieDoc.director }}
        </h2>
      </div>

      <div class="flex flex-col space-y-3">
        <h2 class="text-grey-700 text-base font-bold uppercase">
          Release Date
        </h2>
        <h2 class="text-grey-600 text-xl font-semibold">
          {{ movieDoc.release_date }}
        </h2>
      </div>
    </div>

    <div class="max-w-full">
      <div class="mx-12" v-if="currentstep === 0">
        <div class="max-w-full mx-12 p-2 mt-7 bg-white shadow-2xl rounded">
          <img :src="movieDoc.poster" alt="poster-image" />
        </div>

        <div class="w-full flex items-center justify-center mt-7">
          <Button size="xl" variant="solid" @click="currentstep++"
            >Book Ticket</Button
          >
        </div>

        <div class="flex flex-col space-y-3 mt-16">
          <h2 class="text-grey-700 text-base font-bold uppercase">Synopsis</h2>
          <p class="text-grey-600 text-lg font-normal">
            {{ movieDoc.synopsis }}
          </p>
        </div>
      </div>
      <div v-else-if="currentstep === 1">
        <h2 class="font-medium text-xl my-7 text-grey-900">How many seats?</h2>
        <div class="flex flex-col w-full space-y-5 mt-6">
          <Button
            size="lg"
            :variant="index === bookingData.numberOfSeats ? 'subtle' : 'white'"
            class="shadow-lg"
            v-for="index in 8"
            :key="index"
            @click="setNumberOfSeats(index)"
            >{{ index }}</Button
          >
        </div>
      </div>

      <div v-else-if="currentstep === 2">
        <div class="flex flex-col space-y-4">
          <h2 class="font-medium text-xl my-7 text-grey-900">Date</h2>
          <Input type="date" v-model="bookingData.date" />
        </div>  

        <div class="flex flex-col space-y-4">
          <h2 class="font-medium text-xl my-7 text-grey-900">Cinema & Show</h2>
          <div class="space-y-2">
            <div
              v-for="theatre in Object.keys(theatreShows.data)"
              :key="theatre.name"
              class="bg-white shadow-xl p-4 rounded flex flex-col space-y-4"
            >
              <h3 class="text-sm font-bold text-grey-800">{{ theatre }}</h3>
              <div class="flex flex-row space-x-2">
                <Button
                  @click="bookingData.show = show.name"
                  :key="show.name"
                  v-for="show in theatreShows.data[theatre]"
                  size="sm"
                  :variant= "show.name === bookingData.show ? 'subtle' : 'outline'"
                  >{{ show.start_time }}</Button
                >
              </div>
            </div>

          </div>
        </div>
      </div>

      <div v-else-if="currentstep === 3">
        <h2 class="font-medium text-xl my-7 text-grey-900">Select seats</h2>
        <div>
          <div
            class="flex flex-row"
            v-for="row in Object.keys(seatStructure)"
            :key="row"
          >
            <span
              @click="selectSeat(row, seat[0])"
              v-for="seat in seatStructure[row]"
              :key="seat"
              class="w-6 h-8 m-2 rounded-[2px]"
              :class="[
                seat[1] === 'Available'
                  ? 'bg-blue-300'
                  : seat[1] === 'Selected'
                  ? 'bg-blue-600'
                  : 'bg-gray-300',
                hasSelectedCorrectNumberOfSeats
                  ? 'cursor-not-allowed'
                  : 'cursor-pointer',
              ]"
            >
            </span>
          </div>
        </div>
      </div>

      <div v-else-if="currentstep === 4">
        <div class="w-full flex items-center flex-col mt-7">
          <h1 class="text-[110px]">🍿</h1>
          <h2 class="font-medium text-xl mt-7 text-gray-900">
            Enjoy your movie
          </h2>
        </div>
      </div>
    </div>

    <div class="flex flex-row mt-6 space-x-2">
      <Button
        class="mt-5"
        size="lg"
        :variant="subtle"
        v-if="currentstep !== 0 && currentstep !== 4"
        @click="currentstep--"
        >Go Back</Button
      >

      <Button
        v-if="currentstep !== 0 && currentstep !== 4"
        :disabled="!nextButtonEnabled"
        size="lg"
        variant="solid"
        @click="handleNextClick()"
        >Next</Button
      >
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed } from 'vue'
import { createDocumentResource, createListResource } from 'frappe-ui'

const movieName = ref('Lucky Baskhar')

const movieResources = createDocumentResource({
  doctype: 'Movie',
  name: movieName.value,
  onSuccess(doc) {
    console.log(doc)
  },
  auto: true,
})

const theatreShows = createListResource({
  doctype: 'Theatre Show',
  fields: ['theatre', 'start_time', 'name'],
  auto: true,
  transform: (shows) => {
    const groupedShows = {}
    for (const show of shows) {
      if (!groupedShows[show.theatre]) {
        groupedShows[show.theatre] = []
      }
      groupedShows[show.theatre].push(show)
    }
    return groupedShows
  },
})

const movieBooking = createListResource({
  doctype: 'Movie Tickets',
  insert: {},
  onSuccess() {
    currentstep.value++
  },

})

function getSeatStructure(alphabets, numbers) {
  const structure = {}
  for (const alphabet of alphabets) {
    structure[alphabet] = []
    for (const number of numbers) {
      structure[alphabet].push([number, 'Available'])
    }
  }
  return structure
}

const seatStructure = reactive(
  getSeatStructure(['A', 'B', 'C', 'D', 'E'], [1, 2, 3, 4, 5, 6, 7])
)

const today = new Date().toISOString()
const currentstep = ref(0)

const bookingData = reactive({
  numberOfSeats: 1,
  seats: [],
  date: today,
  selectedSeats: [],
  show: null,
})

function setNumberOfSeats(n) {
  bookingData.numberOfSeats = n
}

function selectSeat(row, number) {
  if (hasSelectedCorrectNumberOfSeats.value) {
    return
  }
  const seat = seatStructure[row].find((seat) => seat[0] === number)
  seat[1] = 'Selected'
  bookingData.selectedSeats.push(`${row}${number}`)
}

const hasSelectedCorrectNumberOfSeats = computed(() => {
  return bookingData.selectedSeats.length === bookingData.numberOfSeats
})

const movieDoc = computed(() => movieResoruces.doc)

const nextButtonEnabled = computed(() => {
  if (currentstep.value === 1) {
    return bookingData.numberOfSeats
  }
  if (currentstep.value === 2) {
    return bookingData.date && bookingData.show
  }
  if (currentstep.value === 3) {
    return hasSelecetedCorrectNumberOfSeats.value 
  }
  return false
})

function handleNextClick() {
  if (currentstep.value !== 3) {
    currentstep.value++
    return
  } 

  movieBooking.insert.submit({
    movie: movieDoc.value.name,
    date: bookingData.date,
    show: bookingData.show,
    seat: JSON.stringify(bookingData.selectedSeats),
    number_of_ticket : bookingData.numberOfSeats
  })  
}
</script> 