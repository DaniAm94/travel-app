<script setup>
import { useForm } from 'laravel-precognition-vue-inertia';
import { Head, Link } from '@inertiajs/vue3';
import GeneralLayout from '@/Layouts/GeneralLayout.vue';
import Modal from '@/Components/Modal.vue';
import { ref, onMounted, watch, nextTick } from 'vue';
import PersonalizedButton from '@/Components/PersonalizedButton.vue';
import DangerButton from '@/Components/DangerButton.vue';

const loadGoogleMapsScript = () => {
    return new Promise((resolve, reject) => {
        if (document.getElementById('google-maps-script')) {
            resolve();
            return;
        }
        const script = document.createElement('script');
        script.id = 'google-maps-script';
        script.src = `https://maps.googleapis.com/maps/api/js?key=${import.meta.env.VITE_GOOGLE_MAPS_API_KEY}&libraries=places`;
        script.async = true;
        script.defer = true;
        script.onload = resolve;
        script.onerror = reject;
        document.head.appendChild(script);
    });
};

const props = defineProps({
    day: Object,
    stops: Array
});

const confirmingUserDeletion = ref(false);
const confirmingStopDeletion = ref(false);
const addressInputRef = ref(null);
const mapRef = ref(null);
const map = ref(null);

const showInfo = ref(false);
const showId = ref(null);

const activeMarkerId = ref(null);

const initialStars = ref(5);
const initialStarsRating = ref(0);

const typeForm = ref(null);

const initializeAutocomplete = () => {
    if (addressInputRef.value && addressInputRef.value instanceof HTMLInputElement) {
        // Inizializza l'autocomplete
        const autocomplete = new google.maps.places.Autocomplete(
            addressInputRef.value,
            { types: ['address'] }
        );

        autocomplete.addListener('place_changed', () => {
            const place = autocomplete.getPlace();
            if (place.geometry) {
                // Aggiorna il campo dell'indirizzo e le coordinate
                if (!updateForm.value) {
                    form.address = place.formatted_address;
                    form.latitude = place.geometry.location.lat();
                    form.longitude = place.geometry.location.lng();
                } else {

                    updateForm.value.address = place.formatted_address;
                    updateForm.value.latitude = place.geometry.location.lat();
                    updateForm.value.longitude = place.geometry.location.lng();
                }
            } else {
                console.error('No details available for input: ' + place.name);
            }
        });
    } else {
        console.error('Address input element not found or not an HTMLInputElement.');
    }
};


const markers = ref({});

const initializeMap = () => {
    if (mapRef.value) {
        // Crea una nuova mappa con un centro e uno zoom predefiniti
        map.value = new google.maps.Map(mapRef.value, {
            center: { lat: 0, lng: 0 }, // Centro di default
            zoom: 2, // Zoom di default
        });

        // Crea bounds per includere tutti i marker
        const bounds = new google.maps.LatLngBounds();

        props.stops.forEach(stop => {
            if (stop.latitude && stop.longitude) {
                const position = { lat: parseFloat(stop.latitude), lng: parseFloat(stop.longitude) };

                // Crea un tag a per contenere l'etichetta personalizzata
                const markerLabel = document.createElement('a');
                markerLabel.className = 'custom-marker';
                markerLabel.href = "#" + stop.slug;
                markerLabel.addEventListener('click', () => {
                    openMap(stop.id);
                });

                // Mostra lo status della stop su mappa
                markerLabel.innerHTML = stop.title;
                const statusElement = document.createElement('span');
                statusElement.classList.add('ml-1')
                if (stop.is_completed) {
                    statusElement.innerHTML = '<i class="fas fa-check"></i>';
                    statusElement.classList.add('text-[#227725]');
                    markerLabel.classList.add('is-completed')
                } else {
                    statusElement.innerHTML = '<i class="fas fa-xmark"></i>';
                    statusElement.classList.add('text-[#1E40AF]')

                }
                markerLabel.appendChild(statusElement);

                // Crea un OverlayView personalizzato
                const customMarker = new google.maps.OverlayView();
                customMarker.onAdd = function () {
                    const panes = this.getPanes();
                    panes.overlayImage.appendChild(markerLabel);
                };
                customMarker.draw = function () {
                    const overlayProjection = this.getProjection();
                    const positionPixel = overlayProjection.fromLatLngToDivPixel(position);
                    markerLabel.style.left = positionPixel.x + 'px';
                    markerLabel.style.top = positionPixel.y + 'px';

                    // Assegna la classe active se questo marker è attivo
                    if (activeMarkerId.value === stop.id) {
                        markerLabel.classList.add('active');
                    } else {
                        markerLabel.classList.remove('active');
                    }
                };
                customMarker.onRemove = function () {
                    markerLabel.parentNode.removeChild(markerLabel);
                };
                customMarker.setPosition = function (latlng) {
                    position = latlng;
                    this.draw();
                };
                customMarker.setMap(map.value);

                // Memorizza il marker
                markers.value[stop.id] = customMarker;

                // Estendi i bounds per includere questa posizione
                bounds.extend(position);
            }
        });

        // Adatta la mappa ai bounds per mostrare tutti i marker
        map.value.fitBounds(bounds);
    }
};




onMounted(() => {
    loadGoogleMapsScript().then(() => {
        // Osserva la visibilità della modale
        watch(() => confirmingUserDeletion.value, (newVal) => {
            if (newVal) {
                nextTick(() => {
                    initializeAutocomplete();
                });
            }
        });

        initializeMap();
    }).catch(error => {
        console.error('Failed to load Google Maps script:', error);
    });
});

// -------  Form creazione stop -----------
const form = useForm('post', '/stops', {
    day_id: props.day.id,
    title: '',
    image: '',
    foods: '',
    address: '',
    latitude: '',
    longitude: '',
});

const submit = () => form.submit({
    preserveScroll: true,
    onSuccess: () => {
        closeModal();
        initializeMap()
    },
});
//  -----------------------------------------

// -------------- Form di modifica stop ----------
const updateForm = ref(null);

const submitUpdateForm = () => updateForm.value.submit({
    preserveScroll: true,
    onSuccess: () => {
        closeModal();
        initializeMap();
    },
});
//  -----------------------------------------

//  ------------     Form per togglare lo status della stop -----------------
// Funzione per inviare la richiesta di aggiornamento dello stato
const toggleStatus = (stopId) => {
    // Inizializza il form con l'ID dello stop
    const toggleForm = useForm('patch', route('stops.toggleStatus', stopId), {

    });

    // Invia la richiesta di aggiornamento
    toggleForm.submit({
        preserveScroll: true,
        onSuccess: () => {
            // Opzionale: esegui eventuali azioni dopo un successo, come aggiornare lo stato localmente
            console.log('Status toggled successfully.');
            initializeMap();
        },
        onError: () => {
            console.error('Failed to toggle status.');
        },
    });
};

//  ---------------------------------------------------------

const openModal = (type, stop) => {

    confirmingUserDeletion.value = true;

    // Passo il tipo di form (create o update)
    typeForm.value = type;

    if (stop) {
        updateForm.value = useForm('put', route('stops.update', stop.id), {
            day_id: props.day.id,
            title: stop.title,
            image: stop.image,
            foods: stop.foods,
            address: stop.address,
            rating: stop.rating,
            latitude: stop.latitude,
            longitude: stop.longitude,
        })
    }
};

const closeModal = () => {
    confirmingUserDeletion.value = false;
    form.reset();
};

// Funzioni per la modale di cancellazione delle tappe
const deleteForm = ref(null);

const submitDeleteForm = () => deleteForm.value.submit({
    preserveScroll: true,
    onSuccess: () => {
        closeDeleteModal();
    },
});

const openDeleteModal = (stop) => {
    deleteForm.value = useForm('delete', route('stops.destroy', stop.id), {
        title: stop.title
    });
    confirmingStopDeletion.value = true;
};

const closeDeleteModal = () => {
    confirmingStopDeletion.value = false;
};

const openMap = (id) => {
    if (showId.value === id) {
        showInfo.value = !showInfo.value;
    } else {
        showInfo.value = true;
        showId.value = id;
    }

    // Aggiorna il marker attivo
    activeMarkerId.value = id;
    if (map.value) {
        // Ridisegna la mappa per aggiornare i marker
        initializeMap();
    }
}

const changeRating = () => {
    // initialStars.value = 5 - initialStarsRating.value;
}
</script>


<template>

    <Head title="Day" />

    <GeneralLayout :isLogged="true">

        <section id="show-day" class="container mx-auto my-24">

            <!-- Title -->
            <h1 class="text-5xl text-center my-6 font-bold">{{ day.title }}</h1>

            <!-- Button Add Stop & Button Go back  -->
            <div class="fixed bottom-5 right-5">
                <button @click="openModal('create')"
                    class="h-12 z-40 w-12 text-white bg-[#75b76f] rounded-full relative group mb-2">
                    <div
                        class="absolute bottom-12 right-6 w-20 bg-[#75b76f] rounded-full hidden group-hover:block z-50">
                        Add stop
                    </div>
                    <font-awesome-icon icon="fas fa-plus" class="fa-lg" />
                </button>
                <Link :href="route('trips.show', day.trip_id)" type="button" as="button"
                    class="h-12 z-40 w-12 text-white bg-[#999999] rounded-full relative group flex items-center justify-center">
                <div
                    class="absolute text-center bottom-12 right-6 w-20 bg-[#999999] rounded-full hidden group-hover:block z-50">
                    Go back
                </div>
                <font-awesome-icon icon="fas fa-arrow-left" class="fa-lg" />
                </Link>
            </div>

            <!-- Mappa -->
            <div id="map" class="rounded-md" ref="mapRef" style="height: 500px; width: 100%;"></div>

            <!-- Tappe (Row) -->
            <div class="flex flex-wrap my-8 -mx-4">

                <!-- Col -->
                <div v-for="stop in stops" :key="stop.id" class="p-4 w-1/3">

                    <!-- Card -->
                    <div :id="stop.slug" :class="{ 'h-[550px] scale-105': showInfo && showId === stop.id }"
                        class="p-4 flex flex-col scale-100 min-w-0 overflow-y-auto overflow-x-hidden no-scrollbar bg-[#FCFCFB] shadow-lg rounded-lg text-center h-[450px] transition-all duration-1000 ease-in-out relative ">

                        <!-- Titolo -->
                        <h2 class="text-3xl font-bold">{{ stop.title }}</h2>

                        <!-- Immagine stop -->
                        <figure class="h-[300px] rounded-lg my-4">
                            <img :src="stop.image ?? 'https://www.svgrepo.com/show/508699/landscape-placeholder.svg'"
                                :alt="stop.title" class="h-full w-full rounded-lg">
                        </figure>

                        <!-- Pulsante info e selezione marker sulla mappa -->
                        <div class="w-full flex justify-between my-4">
                            <button @click="openMap(stop.id)" class="flex items-center gap-1">
                                Info
                                <font-awesome-icon icon="fa-solid fa-angle-down"
                                    :class="{ 'rotate-180': showInfo === true && showId === stop.id }"
                                    class="transition-all duration-100 ease-in-out" />
                            </button>

                            <!-- Bottone per toggolare lo status -->
                            <label class="inline-flex items-center cursor-pointer">
                                <input class="sr-only peer" @change="toggleStatus(stop.id)" :checked="stop.is_completed"
                                    type="checkbox" role="switch" :id="'status-' + stop.id">
                                <div
                                    class="relative w-11 h-6 bg-gray-200 peer-focus:outline-none peer-focus:ring-4 peer-focus:ring-blue-300 dark:peer-focus:ring-blue-800 rounded-full peer dark:bg-gray-700 peer-checked:after:translate-x-full rtl:peer-checked:after:-translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:start-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-5 after:w-5 after:transition-all dark:border-gray-600 peer-checked:bg-blue-600">
                                </div>
                                <span class="ms-3 text-sm font-medium text-gray-700">{{ stop.is_completed ? 'Done' :
                                    'Todo ' }}</span>
                            </label>
                        </div>

                        <!-- Descrizione -->
                        <div v-if="showInfo && showId === stop.id"
                            :class="{ 'h-60 opacity-100': showInfo && showId === stop.id }"
                            class="rounded-lg opacity-0">
                            <!-- Row -->
                            <div class="flex flex-wrap my-4">
                                <!-- Col-8 (Informazioni) -->
                                <div class="w-2/3 flex flex-col items-center justify-center">
                                    <!-- Foods -->
                                    <p v-if="stop.foods" class="mt-2">
                                        <strong>Foods: </strong>{{ stop.foods }}
                                    </p>
                                    <!-- Rating -->
                                    <p v-if="stop.rating !== 0">
                                        <strong>Rating: </strong>
                                        <font-awesome-icon v-for="(rating, i) in stop.rating" :key="i"
                                            icon="fa-solid fa-star" class="text-yellow-600 mt-2 mr-1" />
                                    </p>
                                    <!-- Se non ci sono info -->
                                    <p v-if="!stop.foods && !stop.rating" class="text-xl font-bold mt-4">
                                        There is no information
                                    </p>
                                </div>
                                <!-- Col-4 (Tasto Modifica)-->
                                <div class="w-1/3 flex flex-col items-center justify-center">
                                    <button @click="openModal('update', stop)"
                                        class="px-4 py-2 rounded-lg text-white bg-[#f3a737] mt-2">
                                        Modify
                                    </button>
                                </div>
                            </div>
                        </div>

                        <!-- Bottone per aprire la modale per la cancellazione -->
                        <button
                            class="h-12 w-12 text-white bg-[#00000066] rounded-bl-lg absolute top-0 right-0 hover:bg-red-600 transition-all duration-1000 ease-in-out"
                            @click="openDeleteModal(stop)">
                            <font-awesome-icon icon="fas fa-trash-can" class="fa-lg" />
                        </button>
                    </div>
                </div>
            </div>
        </section>

        <!-- Modale per conferma eliminazione tappa -->
        <Modal :show="confirmingStopDeletion" @close="closeDeleteModal">
            <div class="p-4 text-center">
                <h1 class="text-xl">Do you want to delete this stop: <span class="font-bold">{{ deleteForm.title
                        }}</span>?</h1>
                <PersonalizedButton @click="submitDeleteForm" colorMode="secondary" label="YES" class="me-3" />
                <DangerButton @click="closeDeleteModal">NO</DangerButton>
            </div>
        </Modal>

        <!-- Modale per creazione o modifica tappa -->
        <Modal :show="confirmingUserDeletion" @close="closeModal">
            <form v-if="typeForm === 'create'" @submit.prevent="submit" class="p-6">
                <!-- title  -->
                <div class="my-2">
                    <label for="title" class="text-2xl text-[#684e52] font-bold">Title</label>
                    <input id="title" v-model="form.title" @change="form.validate('title')"
                        class="mt-1 text-lg block h-12 border-gray-300 focus:border-[#684e52] focus:ring-[#684e52] rounded-md shadow-sm w-full" />
                    <div v-if="form.invalid('title')">{{ form.errors.title }}</div>
                </div>

                <!-- image  -->
                <div class="my-2">
                    <label for="image" class="text-2xl text-[#684e52] font-bold">Image</label>
                    <input id="image" v-model="form.image" @change="form.validate('image')"
                        class="mt-1 text-lg block h-12 border-gray-300 focus:border-[#684e52] focus:ring-[#684e52] rounded-md shadow-sm w-full" />
                    <div v-if="form.invalid('image')">{{ form.errors.image }}</div>
                </div>

                <!-- food  -->
                <div class="my-2">
                    <label for="foods" class="text-2xl text-[#684e52] font-bold">Foods</label>
                    <textarea cols="30" rows="10" id="foods" v-model="form.foods" @change="form.validate('foods')"
                        class="w-full border-gray-300 focus:border-[#684e52] focus:ring-[#684e52] rounded-md shadow-sm"></textarea>
                    <div v-if="form.invalid('foods')">{{ form.errors.foods }}</div>
                </div>

                <!-- address  -->
                <div class="my-2">
                    <label for="address-input" class="text-2xl text-[#684e52] font-bold">Address</label>
                    <input id="address-input" ref="addressInputRef" v-model="form.address"
                        @change="form.validate('address')"
                        class="mt-1 text-lg block h-12 border-gray-300 focus:border-[#684e52] focus:ring-[#684e52] rounded-md shadow-sm w-full" />
                    <div v-if="form.invalid('address')">{{ form.errors.address }}</div>
                </div>

                <input id="latitude" v-model="form.latitude" type="hidden" />
                <input id="longitude" v-model="form.longitude" type="hidden" />

                <div class="flex justify-center">
                    <button
                        class=" inline-flex items-center mt-8 px-4 py-2 border border-transparent rounded-md font-semibold text-xs text-white  uppercase tracking-widest hover:shadow-xl transition ease-in-out duration-150 hover:bg-[#443c3d] bg-[#684e52]"
                        :disabled="form.processing">
                        Create Stop
                    </button>
                </div>
            </form>

            <form v-if="typeForm === 'update'" @submit.prevent="submitUpdateForm" class="p-6">

                <!-- title  -->
                <div class="my-2">
                    <label for="title" class="text-2xl text-[#684e52] font-bold">Title</label>
                    <input id="title" v-model="updateForm.title" @change="updateForm.validate('title')"
                        class="mt-1 text-lg block h-12 border-gray-300 focus:border-[#684e52] focus:ring-[#684e52] rounded-md shadow-sm w-full" />
                    <div v-if="updateForm.invalid('title')">{{ updateForm.errors.title }}</div>
                </div>

                <!-- image  -->
                <div class="my-2">
                    <label for="image" class="text-2xl text-[#684e52] font-bold">Image</label>
                    <input id="image" v-model="updateForm.image" @change="updateForm.validate('image')"
                        class="mt-1 text-lg block h-12 border-gray-300 focus:border-[#684e52] focus:ring-[#684e52] rounded-md shadow-sm w-full" />
                    <div v-if="updateForm.invalid('image')">{{ updateForm.errors.image }}</div>
                </div>

                <!-- food  -->
                <div class="my-2">
                    <label for="foods" class="text-2xl text-[#684e52] font-bold">Foods</label>
                    <textarea cols="30" rows="10" id="foods" v-model="updateForm.foods"
                        @change="updateForm.validate('foods')"
                        class="w-full border-gray-300 focus:border-[#684e52] focus:ring-[#684e52] rounded-md shadow-sm"></textarea>
                    <div v-if="updateForm.invalid('foods')">{{ updateForm.errors.foods }}</div>
                </div>

                <!-- Rating -->
                <div class="my-2 flex flex-wrap">

                    <!-- Col-6 -->
                    <div class="w-1/2">
                        <label for="address-input" class="text-2xl text-[#684e52] font-bold">Rating</label>
                        <input id="address-input" type="number" min="0" max="5" @change="changeRating"
                            v-model="updateForm.rating"
                            class="mt-1 text-lg block h-12 border-gray-300 focus:border-[#684e52] focus:ring-[#684e52] rounded-md shadow-sm w-full" />
                        <div v-if="updateForm.invalid('rating')">{{ updateForm.errors.rating }}</div>
                    </div>

                    <!-- Col-6 -->
                    <div class="w-1/2 flex justify-center items-end">

                        <font-awesome-icon v-for="(star, i) in initialStars" icon="fas fa-star"
                            class="fa-2x mb-2 text-gray-300"
                            :class="{ 'text-yellow-600': i + 1 <= updateForm.rating }" />

                    </div>
                </div>

                <!-- address  -->
                <div class="my-2">
                    <label for="address-input" class="text-2xl text-[#684e52] font-bold">Address</label>
                    <input id="address-input" ref="addressInputRef" v-model="updateForm.address"
                        @change="updateForm.validate('address')"
                        class="mt-1 text-lg block h-12 border-gray-300 focus:border-[#684e52] focus:ring-[#684e52] rounded-md shadow-sm w-full" />
                    <div v-if="updateForm.invalid('address')">{{ updateForm.errors.address }}</div>
                </div>

                <input id="latitude" v-model="updateForm.latitude" type="hidden" />
                <input id="longitude" v-model="updateForm.longitude" type="hidden" />

                <div class="flex justify-center">
                    <!-- Create -->
                    <button v-if="typeForm === 'create'"
                        class=" inline-flex items-center mt-8 px-4 py-2 border border-transparent rounded-md font-semibold text-xs text-white  uppercase tracking-widest hover:shadow-xl transition ease-in-out duration-150 hover:bg-[#443c3d] bg-[#684e52]"
                        :disabled="form.processing">
                        Create Stop
                    </button>

                    <!-- Update -->
                    <button v-if="typeForm === 'update'"
                        class=" inline-flex items-center mt-8 px-4 py-2 border border-transparent rounded-md font-semibold text-xs text-white  uppercase tracking-widest hover:shadow-xl transition ease-in-out duration-150 hover:bg-[#443c3d] bg-[#684e52]"
                        :disabled="updateForm.processing">
                        Update Stop
                    </button>
                </div>
            </form>
        </Modal>
    </GeneralLayout>
</template>

<style>
.custom-marker {
    text-transform: capitalize;
    background-color: #DBEAFE;
    color: #1E40AF;
    border: 1px solid #1E40AF;
    padding: 5px;
    border-radius: 50px;
    text-align: center;
    font-weight: bold;
    font-size: 12px;
    white-space: nowrap;
    transform: translate(-50%, -100%);
    position: absolute;

    &.active {
        padding: 7px;
        font-size: 14px;
        border-width: 3px;
    }
}

.custom-marker.is-completed {
    background-color: #d7ffd8;
    color: #227725;
    border: 1px solid #227725;

    &.active {
        padding: 7px;
        font-size: 14px;
        border-width: 3px;
    }
}
</style>