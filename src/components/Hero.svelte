<script>
  import { onMount } from 'svelte';
  
  let loaded = false;
  
  onMount(() => {
    setTimeout(() => loaded = true, 100);
  });
</script>

<section id="home" class="relative h-screen flex items-center justify-center overflow-hidden">
  <!-- Background Image -->
  <div class="absolute inset-0 bg-[url('https://images.unsplash.com/photo-1602216056096-3b40cc0c9944?q=80&w=2070')] bg-cover bg-center">
    <div class="absolute inset-0 bg-gradient-to-b from-black/50 via-black/30 to-black/60"></div>
  </div>

  <!-- Content -->
  <div class="relative z-10 text-center px-4 max-w-4xl mx-auto">
    <h1 class="text-5xl md:text-7xl font-bold text-white mb-4 transition-all duration-1000 {loaded ? 'opacity-100 translate-y-0' : 'opacity-0 -translate-y-10'}">
      Discover <span class="text-kerala-gold">Kerala</span>
    </h1>
    <p class="text-2xl md:text-3xl text-white/90 font-light mb-6 transition-all duration-1000 delay-200 {loaded ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-10'}">God's Own Country</p>
    <p class="text-lg text-white/80 mb-8 max-w-2xl mx-auto transition-all duration-1000 delay-400 {loaded ? 'opacity-100' : 'opacity-0'}">
      Experience the enchanting backwaters, lush green hills, pristine beaches, and rich cultural heritage of Kerala.
    </p>
    <div class="flex flex-col sm:flex-row gap-4 justify-center transition-all duration-1000 delay-700 {loaded ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-10'}">
      <a href="#destinations" class="px-8 py-4 bg-kerala-gold text-white rounded-lg font-semibold hover:bg-yellow-600 transition transform hover:scale-110 hover:-rotate-1">
        Explore Destinations
      </a>
      <a href="#experiences" class="px-8 py-4 border-2 border-white text-white rounded-lg font-semibold hover:bg-white hover:text-kerala-green transition transform hover:scale-110 hover:rotate-1">
        Plan Your Trip
      </a>
    </div>
  </div>

  <!-- Scroll Indicator -->
  <div class="absolute bottom-8 left-1/2 -translate-x-1/2 animate-bounce">
    <svg class="w-8 h-8 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 14l-7 7m0 0l-7-7m7 7V3" />
    </svg>
  </div>
</section>
