<template>
  <div class="">
    <h1 class="text-grey-900 font-bold text-[32px]">Lucky Basker</h1>
    <div class="mt-11 flex flex-row items-center justify-between">
      <div class="flex flex-col space-y-3">
        <h2 class="text-grey-700 text-base font-bold uppercase">Director</h2>
        <h2 class="text-grey-600 text-xl font-semibold">Venky Atluri</h2>
      </div>

      <div class="flex flex-col space-y-3">
        <h2 class="text-grey-700 text-base font-bold uppercase">
          Release Date
        </h2>
        <h2 class="text-grey-600 text-xl font-semibold">9 Jan,2025</h2>
      </div>
    </div>

    <div class="max-w-full">
      <div class="mx-12" v-if="currentstep === 0">
        <div class="max-w-full mx-12 p-2 mt-7 bg-white shadow-2xl rounded">
          <img
            src="https://image.tmdb.org/t/p/original/a47JQFl9L7VDa79tEvnTOJe0rPa.jpg"
            alt="poster-image"
          />
        </div>

        <div class="w-full flex items-center justify-center mt-7">
          <Button size="xl" variant="solid" @click="currentstep++"
            >Book Ticket</Button
          >
        </div>

        <div class="flex flex-col space-y-3 mt-16">
          <h2 class="text-grey-700 text-base font-bold uppercase">Synopsis</h2>
          <p class="text-grey-600 text-lg font-normal">
            The film was officially announced in May 2023 and its title followed
            in July. Principal photography commenced in October, predominantly
            in Hyderabad. Lucky Baskhar has music composed by G. V. Prakash
            Kumar, cinematography handled by Nimish Ravi and editing by Naveen
            Nooli.[6][7][8]
          </p>
        </div>
      </div>
      <div v-else-if="currentstep === 1">
        <h2 class="font-medium text-xl my-7 text-grey-900">How many seats?</h2>
        <div class="flex flex-col w-full space-y-5 mt-6">
          <Button
            size="lg"
            :variant="index === bookingData.numberOfSeats ? 'subtule' : 'white'"
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
          <div class="space-y-4">
            <div class="bg-white shadow-xl p-4 rounded flex flex-col space-y-4">
              <h3 class="text-sm font-bold text-grey-800">Start talkies</h3>
              <div class="flex flex-row space-x-2">
                <Button size="sm" variant="outline">12:30 PM</Button>
                <Button size="sm" variant="subtle">03:30 PM</Button>
              </div>
            </div>

            <div class="bg-white shadow-xl p-4 rounded flex flex-col space-y-4">
              <h3 class="text-sm font-bold text-grey-800">Anupama Theatre</h3>
              <div class="flex flex-row space-x-2">
                <Button size="sm" variant="outline">11:30 AM</Button>
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
              :class="
                seat[1] === 'Available'
                  ? 'bg-blue-300'
                  : seat[1] === 'Selected'
                  ? 'bg-blue-600'
                  : 'bg-gray-300'
              "
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
        :variant="subtule"
        v-if="currentstep !== 0 && currentstep !== 4"
        @click="currentstep--"
        >Go Back</Button
      >

      <Button
        v-if="currentstep !== 0 && currentstep !== 4"
        size="lg"
        variant="solid"
        @click="currentstep++"
        >Next</Button
      >
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'

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
const today = new Date().toISOString
const currentstep = ref(0)
const bookingData = reactive({
  numberOfSeats: 0,
  seats: [],
  date: today,
  selectedSeats: [],
})

function setNumberOfSeats(n) {
  bookingData.numberOfSeats = n
}
function selectSeat(row, number) {
  const seat = seatStructure[row].find(
    (seat) => seat[0] === number && seat[1] === 'Available'
  )
  if (seat) {
    seat[1] = 'Selected'
    bookingData.selectedSeats.push(`${row}${number}`)
  }
}
</script> 