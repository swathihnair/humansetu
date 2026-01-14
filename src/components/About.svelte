<script>
  import { onMount } from 'svelte';
  
  let visible = false;
  let section;
  
  onMount(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach(entry => {
          if (entry.isIntersecting) visible = true;
        });
      },
      { threshold: 0.2 }
    );
    
    if (section) observer.observe(section);
    return () => observer.disconnect();
  });
</script>

<section id="about" class="py-20 bg-white overflow-hidden" bind:this={section}>
  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <div class="grid grid-cols-1 lg:grid-cols-2 gap-12 items-center">
      <div class="transition-all duration-1000 {visible ? 'opacity-100 translate-x-0' : 'opacity-0 -translate-x-20'}">
        <h2 class="text-4xl font-bold text-kerala-green mb-6">Why Visit Kerala?</h2>
        <p class="text-gray-600 mb-6 leading-relaxed">
          Kerala, nestled on India's southwestern coast, is a paradise of natural beauty and cultural richness. 
          From the serene backwaters of Alleppey to the misty hills of Munnar, every corner tells a story.
        </p>
        <div class="space-y-4">
          {#each [
            { title: 'UNESCO Heritage Sites', desc: 'Western Ghats biodiversity hotspot' },
            { title: '100% Literacy Rate', desc: "India's most educated state" },
            { title: 'Ayurveda Capital', desc: 'World-renowned wellness destination' }
          ] as item, i}
            <div 
              class="flex items-start gap-4 transition-all duration-700 {visible ? 'opacity-100 translate-x-0' : 'opacity-0 -translate-x-10'}"
              style="transition-delay: {300 + i * 150}ms"
            >
              <div class="w-12 h-12 bg-kerala-gold/20 rounded-lg flex items-center justify-center flex-shrink-0 transform transition-transform hover:scale-110 hover:rotate-6">
                <span class="text-kerala-gold text-xl">✓</span>
              </div>
              <div>
                <h4 class="font-semibold text-kerala-green">{item.title}</h4>
                <p class="text-gray-600 text-sm">{item.desc}</p>
              </div>
            </div>
          {/each}
        </div>
      </div>
      <div class="relative transition-all duration-1000 delay-300 {visible ? 'opacity-100 translate-x-0 rotate-0' : 'opacity-0 translate-x-20 rotate-6'}">
        <img src="https://images.unsplash.com/photo-1609340757984-7e8f0b0c4c3e?q=80&w=800" alt="Kerala Culture" class="rounded-2xl shadow-2xl transform transition-transform hover:scale-105 duration-500">
        <div class="absolute -bottom-6 -left-6 bg-kerala-gold text-white p-6 rounded-xl shadow-lg transform transition-all duration-500 hover:scale-110 hover:-rotate-3 {visible ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-10'}" style="transition-delay: 600ms">
          <p class="text-3xl font-bold">50+</p>
          <p class="text-sm">Years of Tourism Excellence</p>
        </div>
      </div>
    </div>
  </div>
</section>
