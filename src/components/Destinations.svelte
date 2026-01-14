<script>
  import { onMount } from 'svelte';
  
  let visible = false;
  let section;
  
  const destinations = [
    {
      name: 'Alleppey Backwaters',
      description: 'Cruise through serene backwaters on traditional houseboats',
      image: 'https://images.unsplash.com/photo-1593693411515-c20261bcad6e?q=80&w=800'
    },
    {
      name: 'Munnar Hills',
      description: 'Explore misty tea plantations and rolling green hills',
      image: 'https://images.unsplash.com/photo-1595815771614-ade9d652a65d?q=80&w=800'
    },
    {
      name: 'Kovalam Beach',
      description: 'Relax on pristine beaches with golden sands',
      image: 'https://images.unsplash.com/photo-1590050752117-238cb0fb12b1?q=80&w=800'
    },
    {
      name: 'Wayanad Wildlife',
      description: 'Discover exotic wildlife in lush forest reserves',
      image: 'https://images.unsplash.com/photo-1585409677983-0f6c41ca9c3b?q=80&w=800'
    }
  ];
  
  onMount(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach(entry => {
          if (entry.isIntersecting) {
            visible = true;
          }
        });
      },
      { threshold: 0.2 }
    );
    
    if (section) observer.observe(section);
    return () => observer.disconnect();
  });
</script>

<section id="destinations" class="py-20 bg-gray-50 overflow-hidden" bind:this={section}>
  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <div class="text-center mb-16 transition-all duration-700 {visible ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-10'}">
      <h2 class="text-4xl font-bold text-kerala-green mb-4">Popular Destinations</h2>
      <p class="text-gray-600 max-w-2xl mx-auto">Explore the most breathtaking locations Kerala has to offer</p>
    </div>

    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-8">
      {#each destinations as dest, i}
        <div 
          class="group relative overflow-hidden rounded-2xl shadow-lg cursor-pointer transition-all duration-700
            {visible ? 'opacity-100 translate-y-0 rotate-0' : 'opacity-0 translate-y-32 -rotate-6'}
            hover:-translate-y-4 hover:shadow-2xl hover:rotate-2"
          style="transition-delay: {i * 150}ms"
        >
          <img src={dest.image} alt={dest.name} class="w-full h-80 object-cover group-hover:scale-125 transition duration-700">
          <div class="absolute inset-0 bg-gradient-to-t from-black/80 via-black/20 to-transparent group-hover:from-kerala-green/90 transition duration-500"></div>
          <div class="absolute bottom-0 left-0 right-0 p-6 transform group-hover:-translate-y-2 transition duration-300">
            <h3 class="text-xl font-bold text-white mb-2">{dest.name}</h3>
            <p class="text-white/80 text-sm group-hover:text-white transition">{dest.description}</p>
            <span class="inline-block mt-3 text-kerala-gold opacity-0 group-hover:opacity-100 transform translate-y-4 group-hover:translate-y-0 transition duration-300">
              Explore →
            </span>
          </div>
        </div>
      {/each}
    </div>
  </div>
</section>
