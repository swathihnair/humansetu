<script>
  import { onMount } from 'svelte';
  
  let visible = false;
  let section;
  let formData = { name: '', email: '', message: '' };
  
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

<section id="contact" class="py-20 bg-gray-50 overflow-hidden" bind:this={section}>
  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <div class="grid grid-cols-1 lg:grid-cols-2 gap-12">
      <div class="transition-all duration-1000 {visible ? 'opacity-100 translate-x-0' : 'opacity-0 -translate-x-20'}">
        <h2 class="text-4xl font-bold text-kerala-green mb-6">Plan Your Journey</h2>
        <p class="text-gray-600 mb-8">Get in touch with our travel experts to create your perfect Kerala itinerary.</p>
        
        <div class="space-y-6">
          {#each [
            { icon: 'M3 5a2 2 0 012-2h3.28a1 1 0 01.948.684l1.498 4.493a1 1 0 01-.502 1.21l-2.257 1.13a11.042 11.042 0 005.516 5.516l1.13-2.257a1 1 0 011.21-.502l4.493 1.498a1 1 0 01.684.949V19a2 2 0 01-2 2h-1C9.716 21 3 14.284 3 6V5z', label: 'Phone', value: '+91 471 232 1132' },
            { icon: 'M3 8l7.89 5.26a2 2 0 002.22 0L21 8M5 19h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z', label: 'Email', value: 'info@keralatourism.org' },
            { icon: 'M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z M15 11a3 3 0 11-6 0 3 3 0 016 0z', label: 'Address', value: 'Thiruvananthapuram, Kerala, India' }
          ] as item, i}
            <div 
              class="flex items-center gap-4 transition-all duration-700 {visible ? 'opacity-100 translate-x-0' : 'opacity-0 -translate-x-10'}"
              style="transition-delay: {200 + i * 150}ms"
            >
              <div class="w-12 h-12 bg-kerala-green rounded-lg flex items-center justify-center transform transition-transform hover:scale-110 hover:rotate-6">
                <svg class="w-6 h-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d={item.icon} />
                </svg>
              </div>
              <div>
                <p class="font-semibold text-kerala-green">{item.label}</p>
                <p class="text-gray-600">{item.value}</p>
              </div>
            </div>
          {/each}
        </div>
      </div>

      <div class="bg-white rounded-2xl shadow-xl p-8 transition-all duration-1000 delay-300 {visible ? 'opacity-100 translate-x-0 rotate-0' : 'opacity-0 translate-x-20 rotate-3'}">
        <form class="space-y-6">
          <div class="transition-all duration-500 {visible ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-4'}" style="transition-delay: 400ms">
            <label class="block text-sm font-medium text-gray-700 mb-2">Your Name</label>
            <input bind:value={formData.name} type="text" class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-kerala-green focus:border-transparent transition-all hover:border-kerala-green" placeholder="Enter your name">
          </div>
          <div class="transition-all duration-500 {visible ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-4'}" style="transition-delay: 500ms">
            <label class="block text-sm font-medium text-gray-700 mb-2">Email Address</label>
            <input bind:value={formData.email} type="email" class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-kerala-green focus:border-transparent transition-all hover:border-kerala-green" placeholder="Enter your email">
          </div>
          <div class="transition-all duration-500 {visible ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-4'}" style="transition-delay: 600ms">
            <label class="block text-sm font-medium text-gray-700 mb-2">Message</label>
            <textarea bind:value={formData.message} rows="4" class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-kerala-green focus:border-transparent transition-all hover:border-kerala-green" placeholder="Tell us about your travel plans"></textarea>
          </div>
          <button type="submit" class="w-full py-4 bg-kerala-green text-white rounded-lg font-semibold hover:bg-green-800 transition-all transform hover:scale-105 hover:-rotate-1 {visible ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-4'}" style="transition-delay: 700ms">
            Send Message
          </button>
        </form>
      </div>
    </div>
  </div>
</section>
