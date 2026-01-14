<script>
  import { onMount } from 'svelte';
  
  let visible = false;
  let section;
  
  const experiences = [
    { icon: '🛶', title: 'Houseboat Stay', desc: 'Float through tranquil backwaters' },
    { icon: '🌿', title: 'Ayurveda Spa', desc: 'Traditional healing therapies' },
    { icon: '🐘', title: 'Wildlife Safari', desc: 'Spot elephants in their habitat' },
    { icon: '💃', title: 'Kathakali Show', desc: 'Classical dance performances' },
    { icon: '🍛', title: 'Kerala Cuisine', desc: 'Authentic Malabar flavors' },
    { icon: '🏄', title: 'Beach Activities', desc: 'Surfing and water sports' }
  ];
  
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

<section id="experiences" class="py-20 bg-kerala-green overflow-hidden" bind:this={section}>
  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <div class="text-center mb-16 transition-all duration-700 {visible ? 'opacity-100 translate-y-0' : 'opacity-0 -translate-y-10'}">
      <h2 class="text-4xl font-bold text-white mb-4">Unique Experiences</h2>
      <p class="text-white/80 max-w-2xl mx-auto">Immerse yourself in the authentic Kerala lifestyle</p>
    </div>

    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
      {#each experiences as exp, i}
        <div 
          class="bg-white/10 backdrop-blur-sm rounded-xl p-8 text-center cursor-pointer
            transition-all duration-700 hover:bg-white/20 hover:scale-105 hover:-rotate-1
            {visible ? 'opacity-100 translate-x-0 scale-100' : (i % 2 === 0 ? 'opacity-0 -translate-x-32 scale-75' : 'opacity-0 translate-x-32 scale-75')}"
          style="transition-delay: {i * 100}ms"
        >
          <span class="text-5xl mb-4 block transform transition-transform duration-300 hover:scale-125 hover:rotate-12">{exp.icon}</span>
          <h3 class="text-xl font-bold text-white mb-2">{exp.title}</h3>
          <p class="text-white/80">{exp.desc}</p>
        </div>
      {/each}
    </div>
  </div>
</section>
