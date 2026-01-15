<script>
  const tripServices = [
    {
      icon: 'ℹ️',
      title: 'Tourist Information Centers',
      description: 'Get local information and assistance',
      color: 'from-blue-500 to-cyan-600'
    },
    {
      icon: '🚗',
      title: 'How to Reach',
      description: 'Transportation options and directions',
      color: 'from-green-500 to-emerald-600'
    },
    {
      icon: '🗺️',
      title: 'Maps',
      description: 'Navigate your destination',
      color: 'from-orange-500 to-red-600'
    },
    {
      icon: '🚨',
      title: 'Emergency Contact',
      description: '24/7 helpline numbers',
      color: 'from-red-500 to-rose-600'
    },
    {
      icon: '💱',
      title: 'Currency',
      description: 'Exchange rates and converters',
      color: 'from-yellow-500 to-amber-600'
    }
  ];

  function getServiceUrl(title) {
    switch(title) {
      case 'How to Reach':
        return '/how-to-reach';
      case 'Maps':
        return '/maps';
      case 'Currency':
        return '/currency';
      case 'Tourist Information Centers':
        return '#'; // Update when page is created
      case 'Emergency Contact':
        return '#'; // Update when page is created
      default:
        return '#';
    }
  }

  function handleServiceClick(service) {
    console.log('Service clicked:', service.title);
    
    // Navigate to specific pages based on service
    switch(service.title) {
      case 'How to Reach':
        console.log('Navigating to /how-to-reach');
        window.location.href = '/how-to-reach';
        break;
      case 'Maps':
        console.log('Navigating to /maps');
        window.location.href = '/maps';
        break;
      case 'Currency':
        console.log('Navigating to /currency');
        window.location.href = '/currency';
        break;
      case 'Tourist Information Centers':
        console.log('Navigate to Tourist Information Centers');
        // Add navigation when page is created
        break;
      case 'Emergency Contact':
        console.log('Navigate to Emergency Contact');
        // Add navigation when page is created
        break;
      default:
        console.log('Clicked:', service.title);
    }
  }
</script>

<!-- Services Grid -->
<div class="p-6">
  <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
    {#each tripServices as service, i}
      <a 
        href={getServiceUrl(service.title)}
        class="group relative bg-white/5 hover:bg-white/10 rounded-2xl p-6 border border-white/10 hover:border-white/30 transition-all duration-300 hover:scale-[1.02] animate-slide-up block"
        style="animation-delay: {i * 100}ms"
      >
        <!-- Icon -->
        <div class="w-16 h-16 bg-gradient-to-br {service.color} rounded-2xl flex items-center justify-center text-3xl mb-4 group-hover:scale-110 transition-transform shadow-lg">
          {service.icon}
        </div>
        
        <!-- Content -->
        <h3 class="font-bold text-white mb-2 text-lg">{service.title}</h3>
        <p class="text-white/60 text-sm">{service.description}</p>
        
        <!-- Hover effect -->
        <div class="absolute inset-0 bg-gradient-to-br {service.color} opacity-0 group-hover:opacity-10 rounded-2xl transition-opacity"></div>
        
        <!-- Arrow indicator -->
        <div class="absolute bottom-4 right-4 w-8 h-8 bg-white/10 rounded-full flex items-center justify-center group-hover:bg-white/20 transition-all group-hover:scale-110">
          <span class="text-white/80">→</span>
        </div>
      </a>
    {/each}
  </div>
</div>

<style>
  @keyframes slideUp {
    from { opacity: 0; transform: translateY(30px); }
    to { opacity: 1; transform: translateY(0); }
  }
  
  .animate-slide-up { animation: slideUp 0.6s ease-out forwards; opacity: 0; }
</style>

