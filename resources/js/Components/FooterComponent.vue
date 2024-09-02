<script>
export default {
    props: {
        logo: String
    }
}

</script>


<template>
    <footer class="bg-[#91635C]">
        <section class="flex p-20 gap-5 container mx-auto flex-col">
            <figure class="h-[100px] w-[100px] flex items-center justify-center">
                <img :src="logo" alt="img">
            </figure>
            <div class="flex justify-between text-white">
                <div>
                    <ul>
                        <li>
                            <h3 class="text-xl font-semibold mb-3">When do you want to depart?</h3>
                        </li>
                        <li><a href="#" class="hover:text-[#fae9b8] transition ease-in-out duration-300">Group trips
                                January</a></li>
                        <li><a href="#" class="hover:text-[#fae9b8] transition ease-in-out duration-300">Group trips
                                February</a></li>
                        <li><a href="#" class="hover:text-[#fae9b8] transition ease-in-out duration-300">Group trips
                                March</a></li>
                        <li><a href="#" class="hover:text-[#fae9b8] transition ease-in-out duration-300">Group trips
                                April</a></li>
                        <li><a href="#" class="hover:text-[#fae9b8] transition ease-in-out duration-300">Group trips
                                May</a></li>
                        <li><a href="#" class="hover:text-[#fae9b8] transition ease-in-out duration-300">Group trips
                                June</a></li>
                    </ul>
                </div>
                <div>
                    <ul>
                        <li>
                            <h3 class="text-xl font-semibold mb-3">Team</h3>
                        </li>
                        <li><a href="https://github.com/emilianogastaldo" target="__blank"
                                class=" hover:text-[#fae9b8] transition ease-in-out duration-300">Emiliano Gastaldo</a>
                        </li>
                        <li><a href="https://github.com/DaniAm94" target="__blank"
                                class=" hover:text-[#fae9b8] transition ease-in-out duration-300">Danilo Amico</a></li>
                        <li><a href="https://github.com/ariannacostanzo"
                                class=" hover:text-[#fae9b8] transition ease-in-out duration-300">Arianna Costanzo</a>
                        </li>
                        <li><a href="https://github.com/SantiGalvan" target="__blank"
                                class=" hover:text-[#fae9b8] transition ease-in-out duration-300">Santiago Galvan</a>
                        </li>
                        <li><a href="https://github.com/GioacchinoArgo" target="__blank"
                                class=" hover:text-[#fae9b8] transition ease-in-out duration-300">Gioacchino Argo</a>
                        </li>
                    </ul>
                </div>
                <div>
                    <ul>
                        <li class="text-xl font-semibold mb-3">Useful Info</li>
                        <li><a href="#" class=" hover:text-[#fae9b8] transition ease-in-out duration-300">FAQ</a></li>
                        <li><a href="#" class=" hover:text-[#fae9b8] transition ease-in-out duration-300">Contact Us</a>
                        </li>
                        <li><a href="#" class=" hover:text-[#fae9b8] transition ease-in-out duration-300">General
                                Conditions of Sale for Tourist Packages</a></li>
                        <li><a href="#" class=" hover:text-[#fae9b8] transition ease-in-out duration-300">Terms and
                                Conditions</a></li>
                    </ul>
                </div>
                <div>
                    <h3 class="text-xl font-semibold mb-3 text-center">Social</h3>
                    <div class="flex gap-5">
                        <ul>
                            <li class="mb-2">
                                <a target="_blank" href="https://www.youtube.com/watch?v=dQw4w9WgXcQ">
                                    <font-awesome-icon icon="fab fa-youtube" target="__blank"
                                        class="fa-2x hover:text-[#fae9b8] transition ease-in-out duration-300" />
                                </a>
                            </li>
                            <li class="mb-2">
                                <a target="_blank" href="https://www.youtube.com/watch?v=dQw4w9WgXcQ">
                                    <font-awesome-icon icon="fab fa-twitter" target="__blank"
                                        class="fa-2x hover:text-[#fae9b8] transition ease-in-out duration-300" />
                                </a>
                            </li>
                        </ul>
                        <ul>
                            <li class="mb-2 text-center">
                                <a target="_blank" href="https://www.youtube.com/watch?v=dQw4w9WgXcQ">
                                    <font-awesome-icon icon="fab fa-instagram" target="__blank"
                                        class="fa-2x hover:text-[#fae9b8] transition ease-in-out duration-300" />
                                </a>
                            </li>
                            <li class="mb-2">
                                <a target="_blank" href="https://www.youtube.com/watch?v=dQw4w9WgXcQ">
                                    <font-awesome-icon icon="fab fa-facebook" target="__blank"
                                        class="fa-2x hover:text-[#fae9b8] transition ease-in-out duration-300" />
                                </a>
                            </li>
                        </ul>
                    </div>
                </div>
            </div>
        </section>
    </footer>
</template>