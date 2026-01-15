<script>
  import { onMount } from 'svelte';
  
  let selectedPlace = '';
  let userName = '';
  let userEmail = '';
  let isLoading = true;
  let showCards = false;
  let hosts = [];
  let selectedHost = null;
  let showHostModal = false;
  let isBooking = false;
  let bookingSuccess = false;
  
  // Host API URL
  const HOST_API_URL = 'https://script.google.com/macros/s/AKfycbyMJOX6SNcCuExmI_98GlgFRBvHfSxAXS-7DwWgxcXx1ZCOBTex0Z1ax3oEYqKRZ-LiJQ/exec';
  
  onMount(async () => {
    // Get selected place from localStorage
    selectedPlace = localStorage.getItem('selectedPlace') || 'Kerala';
    
    // Get user info
    const storedUser = localStorage.getItem('keralaUser');
    if (storedUser) {
      const user = JSON.parse(storedUser);
      userName = user.name || 'Traveller';
      userEmail = user.email || '';
    }
    
    // Fetch hosts from the database
    await fetchHosts();
    
    // Show cards after a brief delay
    setTimeout(() => {
      showCards = true;
    }, 500);
  });
  
  async function fetchHosts() {
    try {
      const response = await fetch(HOST_API_URL);
      if (response.ok) {
        const allHosts = await response.json();
        
        console.log('All hosts:', allHosts); // Debug log
        
        // Normalize selected place for matching
        const selectedNormalized = selectedPlace.toLowerCase().trim();
        
        // Extract meaningful keywords
        const placeKeywords = selectedNormalized
          .split(/[\s(),\-]+/)
          .filter(w => w.length > 2 && !['the', 'and', 'for', 'with'].includes(w));
        
        // Filter hosts - check Location, place, AND About You fields
        hosts = allHosts.filter(host => {
          // Get location from various possible field names
          const hostLocation = (host.Location || host.location || host.place || '').toLowerCase().trim();
          const hostAbout = (host['About You'] || host['about you'] || '').toLowerCase();
          const hostName = (host.Name || host.name || '').toLowerCase();
          
          // Check if any keyword matches location OR is mentioned in About You
          return placeKeywords.some(keyword => 
            hostLocation.includes(keyword) || 
            keyword.includes(hostLocation) ||
            hostAbout.includes(keyword)
          ) || hostLocation === selectedNormalized;
        });
        
        console.log(`Found ${hosts.length} hosts in ${selectedPlace}`, hosts);
      }
    } catch (error) {
      console.log('Could not fetch hosts:', error);
    } finally {
      isLoading = false;
    }
  }
  
  // Helper to get host location from various fields
  function getHostLocation(host) {
    return host.Location || host.location || host.place || selectedPlace;
  }
  
  // Helper to get host name
  function getHostName(host) {
    return host.Name || host.name || 'Local Host';
  }
  
  // Helper to get about text
  function getHostAbout(host) {
    return host['About You'] || host['about you'] || '';
  }
  
  // Helper to get activities
  function getHostActivities(host) {
    return host.Activities || host.activities || '';
  }
  
  // Helper to get phone
  function getHostPhone(host) {
    return host['Phone Number'] || host['phone number'] || 'Not available';
  }
  
  // Helper to get email
  function getHostEmail(host) {
    return host.Email || host.email || 'Not available';
  }
  
  function goBack() {
    window.location.href = '/chat';
  }
  
  function goHome() {
    window.location.href = '/';
  }
  
  function contactHost(host) {
    const name = getHostName(host);
    const phone = getHostPhone(host);
    const email = getHostEmail(host);
    alert(`Contacting ${name}!\n\nPhone: ${phone}\nEmail: ${email}`);
  }
  
  function confirmHost(host) {
    selectedHost = host;
    showHostModal = true;
  }
  
  function closeModal() {
    showHostModal = false;
    selectedHost = null;
    bookingSuccess = false;
    isBooking = false;
  }
  
  // Get unique experiences the host offers
  function getHostExperiences(host) {
    const activities = getHostActivities(host);
    if (!activities) return [];
    return activities.split(',').map(a => a.trim()).filter(a => a.length > 0);
  }
  
  // Send booking request
  function sendBookingRequest() {
    if (!selectedHost || !userName) return;
    
    isBooking = true;
    
    // Create booking request
    const bookingRequest = {
      id: Date.now(),
      traveller: userName,
      travellerEmail: userEmail,
      hostName: getHostName(selectedHost),
      hostEmail: getHostEmail(selectedHost),
      hostPhone: getHostPhone(selectedHost),
      location: getHostLocation(selectedHost),
      activities: getHostActivities(selectedHost),
      place: selectedPlace,
      date: new Date().toISOString().split('T')[0],
      status: 'pending',
      createdAt: new Date().toISOString()
    };
    
    // Get existing booking requests from localStorage
    const existingRequests = JSON.parse(localStorage.getItem('bookingRequests') || '[]');
    existingRequests.push(bookingRequest);
    localStorage.setItem('bookingRequests', JSON.stringify(existingRequests));
    
    // Also save to traveller's bookings
    const travellerBookings = JSON.parse(localStorage.getItem('travellerBookings') || '[]');
    travellerBookings.push(bookingRequest);
    localStorage.setItem('travellerBookings', JSON.stringify(travellerBookings));
    
    // Show success briefly then redirect to traveller dashboard
    setTimeout(() => {
      isBooking = false;
      bookingSuccess = true;
      
      // Auto redirect to traveller dashboard after 1.5 seconds
      setTimeout(() => {
        window.location.href = '/traveller-dashboard';
      }, 1500);
    }, 1500);
  }
  
  // Go to traveller dashboard
  function goToTravellerDashboard() {
    window.location.href = '/traveller-dashboard';
  }
</script>

<div class="min-h-screen relative overflow-hidden">
  <!-- Kerala Themed Background with Zoom Effect -->
  <div class="fixed inset-0 bg-zoom" style="z-index: 0;">
    <div 
      class="absolute inset-0 bg-cover bg-center bg-no-repeat kerala-bg-1"
      style="background-image: url('https://images.unsplash.com/photo-1595815771614-ade9d652a65d?q=80&w=1920');"
    ></div>
    <div 
      class="absolute inset-0 bg-cover bg-center bg-no-repeat kerala-bg-2 opacity-30"
      style="background-image: url('https://images.unsplash.com/photo-1590050752117-238cb0fb12b1?q=80&w=1920');"
    ></div>
  </div>
  <div class="fixed inset-0 bg-gradient-to-br from-kerala-green/50 via-transparent to-kerala-gold/30" style="z-index: 1;"></div>
  <div class="fixed inset-0 particles-container" style="z-index: 2;"></div>
  
  <!-- Main Content -->
  <div class="relative z-10 transition-all duration-1000 {showCards ? 'opacity-100' : 'opacity-0'}">
    <!-- Header -->
    <header class="bg-black/50 backdrop-blur-md border-b border-white/10">
      <div class="max-w-7xl mx-auto px-4 py-4 flex items-center justify-between">
        <button on:click={goHome} class="text-2xl font-bold text-white hover:text-kerala-gold transition">
          Kerala<span class="text-kerala-gold">Tourism</span>
        </button>
        <button on:click={goBack} class="px-4 py-2 text-white border border-white/30 rounded-lg hover:bg-white/10 transition">
          ← Back to Chat
        </button>
      </div>
    </header>
    
    <!-- Main Content -->
    <main class="max-w-7xl mx-auto px-4 py-8">
      <div class="text-center mb-12">
        <h1 class="text-4xl font-bold text-white mb-4 drop-shadow-lg">
          🎯 Perfect Hosts in <span class="text-kerala-gold">{selectedPlace}</span>
        </h1>
        <p class="text-white/80 text-lg max-w-2xl mx-auto">
          {userName ? `Hey ${userName}! ` : ''}We found local hosts who can give you an authentic experience.
        </p>
      </div>
      
      {#if hosts.length === 0 && !isLoading}
        <div class="text-center py-20 bg-black/30 backdrop-blur-sm rounded-2xl">
          <div class="text-6xl mb-4">🏠</div>
          <h2 class="text-2xl font-bold text-white mb-2">No hosts found yet</h2>
          <p class="text-white/60 mb-6">We're still building our host network in {selectedPlace}.</p>
          <button on:click={goBack} class="px-6 py-3 bg-kerala-green text-white rounded-lg font-semibold hover:bg-green-800 transition">
            Explore Other Places
          </button>
        </div>
      {:else}
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
          {#each hosts as host, i}
            <div 
              class="card-drop bg-white/95 backdrop-blur-md rounded-2xl overflow-hidden border-2 border-white/30 hover:border-kerala-gold/50 transition transform hover:-translate-y-2 hover:shadow-2xl"
              style="animation-delay: {i * 0.2}s"
            >
              <div class="bg-gradient-to-r from-kerala-green to-kerala-gold p-6 text-white">
                <div class="flex items-center gap-4">
                  <div class="w-16 h-16 bg-white/20 rounded-full flex items-center justify-center text-3xl">
                    🏠
                  </div>
                  <div>
                    <h3 class="text-xl font-bold">{getHostName(host)}</h3>
                    <p class="text-white/80">📍 {getHostLocation(host)}</p>
                  </div>
                </div>
              </div>
              
              <div class="p-6 bg-white">
                {#if getHostAbout(host)}
                  <div class="mb-4">
                    <h4 class="text-sm font-semibold text-kerala-green mb-1">About</h4>
                    <p class="text-gray-700 line-clamp-3">{getHostAbout(host)}</p>
                  </div>
                {/if}
                
                {#if getHostActivities(host)}
                  <div class="mb-4">
                    <h4 class="text-sm font-semibold text-kerala-green mb-2">Activities</h4>
                    <div class="flex flex-wrap gap-2">
                      {#each getHostActivities(host).split(',').slice(0, 4) as activity}
                        <span class="px-3 py-1 bg-kerala-green/10 text-kerala-green text-xs rounded-full">
                          {activity.trim()}
                        </span>
                      {/each}
                    </div>
                  </div>
                {/if}
                
                <button 
                  on:click={() => confirmHost(host)}
                  class="w-full py-3 bg-gradient-to-r from-kerala-green to-kerala-gold text-white rounded-lg font-semibold hover:opacity-90 transition"
                >
                  ✓ Confirm Host
                </button>
              </div>
            </div>
          {/each}
        </div>
        
        <div class="text-center mt-12 p-8 bg-black/40 backdrop-blur-md rounded-2xl border border-white/20">
          <h3 class="text-2xl font-bold text-white mb-2">Can't find the right host?</h3>
          <p class="text-white/60 mb-4">Tell us more about your preferences.</p>
          <button on:click={goBack} class="px-6 py-3 bg-kerala-gold text-white rounded-lg font-semibold hover:bg-yellow-600 transition">
            Refine My Search
          </button>
        </div>
      {/if}
    </main>
  </div>
  
  <!-- Host Details Modal -->
  {#if showHostModal && selectedHost}
    <div class="fixed inset-0 z-50 flex items-center justify-center p-4">
      <!-- Backdrop -->
      <div 
        class="absolute inset-0 bg-black/70 backdrop-blur-sm"
        on:click={closeModal}
      ></div>
      
      <!-- Modal Content -->
      <div class="relative bg-white rounded-3xl max-w-lg w-full max-h-[90vh] overflow-y-auto shadow-2xl animate-modal-pop">
        <!-- Header with gradient -->
        <div class="bg-gradient-to-r from-kerala-green to-kerala-gold p-6 rounded-t-3xl">
          <button 
            on:click={closeModal}
            class="absolute top-4 right-4 w-8 h-8 bg-white/20 rounded-full flex items-center justify-center text-white hover:bg-white/30 transition"
          >
            ✕
          </button>
          <div class="flex items-center gap-4">
            <div class="w-20 h-20 bg-white/20 rounded-full flex items-center justify-center text-4xl">
              🏠
            </div>
            <div>
              <h2 class="text-2xl font-bold text-white">{getHostName(selectedHost)}</h2>
              <p class="text-white/80 flex items-center gap-1">
                📍 {getHostLocation(selectedHost)}
              </p>
            </div>
          </div>
        </div>
        
        <!-- Body -->
        <div class="p-6 space-y-6">
          <!-- About Section -->
          {#if getHostAbout(selectedHost)}
            <div>
              <h3 class="text-lg font-bold text-kerala-green mb-2 flex items-center gap-2">
                <span class="text-xl">👤</span> About Me
              </h3>
              <p class="text-gray-700 leading-relaxed">{getHostAbout(selectedHost)}</p>
            </div>
          {/if}
          
          <!-- Activities/Experiences Section -->
          {#if getHostExperiences(selectedHost).length > 0}
            <div>
              <h3 class="text-lg font-bold text-kerala-green mb-3 flex items-center gap-2">
                <span class="text-xl">🌟</span> Experiences You'll Get
              </h3>
              <div class="grid grid-cols-1 gap-2">
                {#each getHostExperiences(selectedHost) as experience}
                  <div class="flex items-center gap-3 p-3 bg-kerala-green/5 rounded-lg">
                    <span class="text-kerala-gold">✦</span>
                    <span class="text-gray-700">{experience}</span>
                  </div>
                {/each}
              </div>
            </div>
          {/if}
          
          <!-- Contact Details Section -->
          <div>
            <h3 class="text-lg font-bold text-kerala-green mb-3 flex items-center gap-2">
              <span class="text-xl">📞</span> Contact Details
            </h3>
            <div class="space-y-3">
              <div class="flex items-center gap-3 p-3 bg-gray-50 rounded-lg">
                <span class="text-2xl">📱</span>
                <div>
                  <p class="text-xs text-gray-500">Phone Number</p>
                  <p class="text-gray-800 font-medium">{getHostPhone(selectedHost)}</p>
                </div>
              </div>
              <div class="flex items-center gap-3 p-3 bg-gray-50 rounded-lg">
                <span class="text-2xl">✉️</span>
                <div>
                  <p class="text-xs text-gray-500">Email Address</p>
                  <p class="text-gray-800 font-medium">{getHostEmail(selectedHost)}</p>
                </div>
              </div>
            </div>
          </div>
          
          <!-- Action Buttons -->
          <div class="flex gap-3 pt-4">
            <a 
              href="tel:{getHostPhone(selectedHost)}"
              class="flex-1 py-3 bg-kerala-green text-white rounded-xl font-semibold hover:bg-green-800 transition flex items-center justify-center gap-2"
            >
              📞 Call Now
            </a>
            <a 
              href="mailto:{getHostEmail(selectedHost)}?subject=Kerala Trip Inquiry&body=Hi {getHostName(selectedHost)}, I found you on Kerala Tourism and I'm interested in visiting {getHostLocation(selectedHost)}."
              class="flex-1 py-3 bg-kerala-gold text-white rounded-xl font-semibold hover:bg-yellow-600 transition flex items-center justify-center gap-2"
            >
              ✉️ Send Email
            </a>
          </div>
          
          <!-- Confirm Booking Button -->
          {#if !bookingSuccess}
            <button 
              on:click={sendBookingRequest}
              disabled={isBooking}
              class="w-full py-4 mt-4 bg-gradient-to-r from-kerala-green to-kerala-gold text-white rounded-xl font-bold text-lg hover:opacity-90 transition disabled:opacity-70 flex items-center justify-center gap-2"
            >
              {#if isBooking}
                <div class="w-5 h-5 border-2 border-white border-t-transparent rounded-full animate-spin"></div>
                Sending Request...
              {:else}
                ✓ Confirm Booking Request
              {/if}
            </button>
          {:else}
            <div class="mt-4 p-4 bg-green-50 border border-green-200 rounded-xl text-center">
              <div class="text-4xl mb-2">🎉</div>
              <h4 class="text-lg font-bold text-green-700">Booking Request Sent!</h4>
              <p class="text-green-600 text-sm mt-1">Redirecting to your dashboard...</p>
              <div class="mt-3 flex items-center justify-center gap-2 text-kerala-green">
                <div class="w-4 h-4 border-2 border-kerala-green border-t-transparent rounded-full animate-spin"></div>
                <span class="text-sm">Taking you to My Trips</span>
              </div>
            </div>
          {/if}
        </div>
      </div>
    </div>
  {/if}
</div>

<style>
  .bg-zoom { overflow: hidden; }
  
  .kerala-bg-1 {
    animation: zoomInOut 20s ease-in-out infinite;
  }
  
  .kerala-bg-2 {
    animation: zoomInOut 25s ease-in-out infinite reverse;
  }
  
  @keyframes zoomInOut {
    0%, 100% { transform: scale(1); }
    50% { transform: scale(1.15); }
  }
  
  .particles-container { pointer-events: none; }
  
  .particles-container::before {
    content: '';
    position: absolute;
    width: 100%;
    height: 100%;
    background-image: 
      radial-gradient(2px 2px at 20% 30%, rgba(212, 168, 83, 0.8), transparent),
      radial-gradient(3px 3px at 40% 70%, rgba(255, 255, 255, 0.6), transparent),
      radial-gradient(2px 2px at 60% 20%, rgba(212, 168, 83, 0.7), transparent),
      radial-gradient(3px 3px at 80% 50%, rgba(255, 255, 255, 0.5), transparent);
    animation: floatParticles 15s ease-in-out infinite;
  }
  
  @keyframes floatParticles {
    0%, 100% { transform: translateY(0); opacity: 0.6; }
    50% { transform: translateY(-20px); opacity: 0.8; }
  }
  
  @keyframes cardDrop {
    0% { opacity: 0; transform: translateY(-50px); }
    100% { opacity: 1; transform: translateY(0); }
  }
  
  .card-drop {
    animation: cardDrop 0.6s ease-out forwards;
    opacity: 0;
  }
  
  .line-clamp-3 {
    display: -webkit-box;
    -webkit-line-clamp: 3;
    -webkit-box-orient: vertical;
    overflow: hidden;
  }
  
  @keyframes modalPop {
    0% { opacity: 0; transform: scale(0.9) translateY(20px); }
    100% { opacity: 1; transform: scale(1) translateY(0); }
  }
  
  .animate-modal-pop {
    animation: modalPop 0.3s ease-out forwards;
  }
</style>
