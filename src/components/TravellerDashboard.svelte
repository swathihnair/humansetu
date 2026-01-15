<script>
  import { onMount } from 'svelte';
  
  let userName = '';
  let userEmail = '';
  let bookings = [];
  let isLoading = true;
  let showExperienceModal = false;
  let selectedBooking = null;
  let experienceText = '';
  let experienceRating = 5;
  let experiences = [];
  let showContent = false;
  let confettiActive = false;
  
  onMount(() => {
    const storedUser = localStorage.getItem('keralaUser');
    if (storedUser) {
      const user = JSON.parse(storedUser);
      userName = user.name || 'Traveller';
      userEmail = user.email || '';
      
      if (user.role === 'host') {
        window.location.href = '/host-dashboard';
        return;
      }
    } else {
      window.location.href = '/';
      return;
    }
    
    loadBookings();
    loadExperiences();
    
    setTimeout(() => {
      isLoading = false;
      setTimeout(() => showContent = true, 100);
      
      // Trigger confetti if there's a confirmed booking
      if (bookings.some(b => b.status === 'confirmed')) {
        confettiActive = true;
      }
    }, 800);
    
    const interval = setInterval(loadBookings, 5000);
    return () => clearInterval(interval);
  });
  
  function loadBookings() {
    const allBookings = JSON.parse(localStorage.getItem('bookingRequests') || '[]');
    const prevConfirmed = bookings.some(b => b.status === 'confirmed');
    bookings = allBookings.filter(b => 
      b.traveller === userName || b.travellerEmail === userEmail
    );
    
    // Trigger confetti when booking gets confirmed
    if (!prevConfirmed && bookings.some(b => b.status === 'confirmed')) {
      confettiActive = true;
    }
  }
  
  function loadExperiences() {
    experiences = JSON.parse(localStorage.getItem('travellerExperiences') || '[]')
      .filter(e => e.travellerName === userName || e.travellerEmail === userEmail);
  }
  
  function openExperienceModal(booking) {
    selectedBooking = booking;
    showExperienceModal = true;
  }
  
  function closeExperienceModal() {
    showExperienceModal = false;
    selectedBooking = null;
    experienceText = '';
    experienceRating = 5;
  }
  
  function submitExperience() {
    if (!experienceText.trim() || !selectedBooking) return;
    
    const newExperience = {
      id: Date.now(),
      travellerName: userName,
      travellerEmail: userEmail,
      hostName: selectedBooking.hostName,
      location: selectedBooking.location,
      place: selectedBooking.place,
      rating: experienceRating,
      text: experienceText,
      createdAt: new Date().toISOString()
    };
    
    const allExperiences = JSON.parse(localStorage.getItem('travellerExperiences') || '[]');
    allExperiences.push(newExperience);
    localStorage.setItem('travellerExperiences', JSON.stringify(allExperiences));
    
    // Also mark the trip as ended
    const allBookings = JSON.parse(localStorage.getItem('bookingRequests') || '[]');
    const updatedBookings = allBookings.map(b => 
      b.id === selectedBooking.id ? { ...b, tripEnded: true } : b
    );
    localStorage.setItem('bookingRequests', JSON.stringify(updatedBookings));
    
    experiences = [...experiences, newExperience];
    closeExperienceModal();
    
    // Redirect to chat for new trip planning
    setTimeout(() => {
      window.location.href = '/chat';
    }, 1500);
  }

  function goHome() { window.location.href = '/'; }
  function goToChat() { window.location.href = '/chat'; }
  function logout() {
    localStorage.removeItem('keralaUser');
    window.location.href = '/';
  }
  
  function clearAllBookings() {
    if (confirm('Are you sure you want to clear all bookings? This cannot be undone.')) {
      localStorage.removeItem('bookingRequests');
      localStorage.removeItem('travellerBookings');
      localStorage.removeItem('travellerExperiences');
      bookings = [];
      experiences = [];
      alert('All bookings cleared!');
      window.location.href = '/chat';
    }
  }
  
  function markTripEnded(booking) {
    // Update the booking status in localStorage
    const allBookings = JSON.parse(localStorage.getItem('bookingRequests') || '[]');
    const updatedBookings = allBookings.map(b => 
      b.id === booking.id ? { ...b, tripEnded: true } : b
    );
    localStorage.setItem('bookingRequests', JSON.stringify(updatedBookings));
    
    // Reload bookings
    loadBookings();
    
    // Redirect to chat for new trip planning
    setTimeout(() => {
      window.location.href = '/chat';
    }, 1500);
  }
  
  function getStatusColor(status) {
    switch(status) {
      case 'confirmed': return 'from-green-400 to-emerald-500';
      case 'declined': return 'from-red-400 to-rose-500';
      default: return 'from-yellow-400 to-amber-500';
    }
  }
</script>

<div class="min-h-screen relative overflow-hidden">
  <!-- Kerala Themed Background -->
  <div class="fixed inset-0">
    <div 
      class="absolute inset-0 bg-cover bg-center bg-no-repeat"
      style="background-image: url('https://images.unsplash.com/photo-1602216056096-3b40cc0c9944?q=80&w=1920');"
    ></div>
    <div class="absolute inset-0 bg-gradient-to-br from-kerala-green/90 via-kerala-green/70 to-kerala-gold/50"></div>
  </div>
  
  <!-- Animated Overlay Elements -->
  <div class="fixed inset-0 pointer-events-none overflow-hidden">
    <!-- Floating leaves -->
    {#each Array(15) as _, i}
      <div 
        class="absolute text-2xl animate-leaf-fall"
        style="left: {Math.random() * 100}%; animation-delay: {i * 1.5}s; animation-duration: {10 + Math.random() * 10}s;"
      >🍃</div>
    {/each}
    
    <!-- Glowing orbs -->
    <div class="absolute top-20 left-10 w-64 h-64 bg-kerala-gold/20 rounded-full filter blur-3xl animate-pulse-slow"></div>
    <div class="absolute bottom-20 right-10 w-80 h-80 bg-white/10 rounded-full filter blur-3xl animate-pulse-slow animation-delay-2000"></div>
  </div>

  <!-- Confetti -->
  {#if confettiActive}
    <div class="fixed inset-0 pointer-events-none z-50">
      {#each Array(50) as _, i}
        <div 
          class="absolute w-3 h-3 animate-confetti"
          style="left: {Math.random() * 100}%; background: {['#D4A853', '#2D5A27', '#10B981', '#F59E0B', '#EC4899'][Math.floor(Math.random() * 5)]}; animation-delay: {Math.random() * 2}s; transform: rotate({Math.random() * 360}deg);"
        ></div>
      {/each}
    </div>
  {/if}
  
  <!-- Main Content -->
  <div class="relative z-10">
    <!-- Header -->
    <header class="bg-black/20 backdrop-blur-xl border-b border-white/10">
      <div class="max-w-7xl mx-auto px-4 py-4 flex items-center justify-between">
        <button on:click={goHome} class="text-2xl font-bold text-white hover:scale-105 transition-transform">
          Kerala<span class="text-kerala-gold">Tourism</span>
        </button>
        <div class="flex items-center gap-4">
          <div class="hidden sm:flex items-center gap-2 px-4 py-2 bg-white/10 rounded-full">
            <div class="w-8 h-8 bg-gradient-to-br from-kerala-gold to-amber-600 rounded-full flex items-center justify-center text-white font-bold">
              {userName.charAt(0).toUpperCase()}
            </div>
            <span class="text-white/90">{userName}</span>
          </div>
          <button on:click={logout} class="px-4 py-2 text-white/80 border border-white/20 rounded-full hover:bg-white/10 transition-all hover:scale-105">
            Logout
          </button>
        </div>
      </div>
    </header>
    
    {#if isLoading}
      <div class="flex items-center justify-center h-[80vh]">
        <div class="text-center">
          <div class="relative w-20 h-20 mx-auto mb-6">
            <div class="absolute inset-0 border-4 border-kerala-gold/30 rounded-full"></div>
            <div class="absolute inset-0 border-4 border-transparent border-t-kerala-gold rounded-full animate-spin"></div>
            <div class="absolute inset-2 border-4 border-transparent border-t-white rounded-full animate-spin-reverse"></div>
          </div>
          <p class="text-white/80 text-lg animate-pulse">Loading your adventures...</p>
        </div>
      </div>
    {:else}
      <main class="max-w-7xl mx-auto px-4 py-8 transition-all duration-700 {showContent ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-10'}">
        
        <!-- Hero Banner -->
        {#if bookings.some(b => b.status === 'confirmed')}
          <div class="relative mb-8 rounded-3xl overflow-hidden animate-slide-up shadow-2xl">
            <!-- Animated gradient background -->
            <div class="absolute inset-0 bg-gradient-to-r from-kerala-gold via-amber-500 to-kerala-gold bg-[length:200%_100%] animate-gradient"></div>
            <div class="absolute inset-0 bg-[url('https://images.unsplash.com/photo-1593693411515-c20261bcad6e?q=80&w=1920')] bg-cover bg-center opacity-20"></div>
            
            <div class="relative p-8 md:p-12">
              <div class="flex flex-col md:flex-row items-center gap-6">
                <!-- Animated celebration icon -->
                <div class="relative">
                  <div class="w-28 h-28 bg-white/20 backdrop-blur-md rounded-3xl flex items-center justify-center text-6xl animate-celebration">
                    🎉
                  </div>
                  <!-- Sparkles around icon -->
                  <div class="absolute -top-2 -right-2 text-2xl animate-sparkle">✨</div>
                  <div class="absolute -bottom-1 -left-2 text-xl animate-sparkle animation-delay-500">✨</div>
                  <div class="absolute top-1/2 -right-4 text-lg animate-sparkle animation-delay-1000">⭐</div>
                </div>
                
                <div class="text-center md:text-left flex-1">
                  <!-- Animated title with typewriter effect -->
                  <h1 class="text-3xl md:text-5xl font-bold text-white mb-3">
                    <span class="inline-block animate-word-pop">Let's</span>
                    <span class="inline-block animate-word-pop animation-delay-200">Get</span>
                    <span class="inline-block animate-word-pop animation-delay-400">Ready</span>
                    <span class="inline-block animate-word-pop animation-delay-600">for</span>
                    <span class="inline-block animate-word-pop animation-delay-800">the</span>
                    <span class="inline-block animate-word-pop animation-delay-1000 text-kerala-green">Trip!</span>
                    <span class="inline-block animate-bounce-in animation-delay-1200">🌴</span>
                  </h1>
                  <p class="text-white/90 text-lg md:text-xl animate-fade-in animation-delay-1500">
                    Your host has confirmed your booking. Time to pack your bags! 🧳✨
                  </p>
                </div>
              </div>
              
              <!-- Decorative elements -->
              <div class="absolute -bottom-6 -right-6 text-9xl opacity-20 animate-sway">🌴</div>
              <div class="absolute -bottom-4 right-24 text-7xl opacity-15 animate-sway animation-delay-1000">🌴</div>
              <div class="absolute top-4 right-8 text-4xl opacity-30 animate-float">🦜</div>
            </div>
          </div>
        {:else}
          <div class="relative mb-8 rounded-3xl overflow-hidden bg-gradient-to-r from-white/10 to-white/5 backdrop-blur-md border border-white/20 animate-slide-up">
            <div class="p-8 flex items-center gap-6">
              <div class="w-20 h-20 bg-gradient-to-br from-kerala-gold to-amber-600 rounded-2xl flex items-center justify-center text-4xl shadow-lg shadow-kerala-gold/30 animate-float">
                🧳
              </div>
              <div>
                <h1 class="text-3xl font-bold text-white mb-1">My Trips</h1>
                <p class="text-white/60">Track your bookings and share your experiences</p>
              </div>
            </div>
          </div>
        {/if}

        <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
          <!-- Bookings Section -->
          <div class="lg:col-span-2 space-y-6">
            <div class="bg-white/10 backdrop-blur-md rounded-3xl border border-white/20 overflow-hidden animate-slide-up animation-delay-200">
              <div class="p-6 border-b border-white/10 flex items-center justify-between">
                <h2 class="text-xl font-bold text-white flex items-center gap-3">
                  <span class="w-10 h-10 bg-kerala-gold/20 rounded-xl flex items-center justify-center">📋</span>
                  My Bookings
                </h2>
                <button 
                  on:click={loadBookings}
                  class="px-4 py-2 bg-white/10 text-white/80 rounded-full hover:bg-white/20 transition-all hover:scale-105 flex items-center gap-2"
                >
                  <span class="animate-spin-slow">🔄</span> Refresh
                </button>
              </div>
              
              <div class="p-6">
                {#if bookings.length === 0}
                  <div class="text-center py-16">
                    <div class="w-24 h-24 mx-auto mb-6 bg-white/10 rounded-3xl flex items-center justify-center text-5xl animate-float">
                      🏠
                    </div>
                    <h3 class="text-2xl font-bold text-white mb-2">No bookings yet</h3>
                    <p class="text-white/60 mb-6">Find a local host and start your Kerala adventure!</p>
                    <button on:click={goToChat} class="px-8 py-4 bg-gradient-to-r from-kerala-gold to-amber-500 text-white rounded-2xl font-semibold hover:shadow-lg hover:shadow-kerala-gold/30 transition-all hover:scale-105">
                      🔍 Find Hosts
                    </button>
                  </div>
                {:else}
                  <div class="space-y-4">
                    {#each bookings as booking, i}
                      <div 
                        class="group relative bg-white/5 hover:bg-white/10 rounded-2xl p-6 border border-white/10 hover:border-white/30 transition-all duration-300 hover:scale-[1.02] animate-slide-up"
                        style="animation-delay: {300 + i * 100}ms"
                      >
                        <!-- Status Badge -->
                        <div class="absolute -top-3 -right-3">
                          <div class="px-4 py-1 bg-gradient-to-r {getStatusColor(booking.status)} rounded-full text-white text-sm font-semibold shadow-lg flex items-center gap-1">
                            {#if booking.status === 'confirmed'}
                              <span class="animate-pulse">✓</span> Confirmed
                            {:else if booking.status === 'declined'}
                              ✗ Declined
                            {:else}
                              <span class="animate-spin-slow">⏳</span> Pending
                            {/if}
                          </div>
                        </div>
                        
                        <div class="flex items-start gap-4">
                          <div class="w-16 h-16 bg-gradient-to-br from-kerala-green to-emerald-600 rounded-2xl flex items-center justify-center text-3xl shadow-lg group-hover:scale-110 transition-transform">
                            🏠
                          </div>
                          <div class="flex-1">
                            <h3 class="text-xl font-bold text-white mb-1">{booking.hostName}</h3>
                            <p class="text-white/70 flex items-center gap-2 mb-2">
                              <span>📍</span> {booking.location || booking.place}
                            </p>
                            {#if booking.activities}
                              <div class="flex flex-wrap gap-2 mb-3">
                                {#each booking.activities.split(',').slice(0, 3) as activity}
                                  <span class="px-3 py-1 bg-white/10 text-white/80 text-xs rounded-full">
                                    {activity.trim()}
                                  </span>
                                {/each}
                              </div>
                            {/if}
                            <p class="text-white/40 text-sm">📅 {new Date(booking.createdAt).toLocaleDateString()}</p>
                          </div>
                        </div>
                        
                        {#if booking.status === 'confirmed'}
                          <div class="mt-4 p-4 bg-emerald-500/20 rounded-xl border border-emerald-500/30">
                            <h4 class="font-semibold text-emerald-400 mb-3 flex items-center gap-2">
                              <span class="animate-pulse">🎉</span> Booking Confirmed! Contact your host:
                            </h4>
                            <div class="flex flex-wrap gap-3 mb-4">
                              <a href="tel:{booking.hostPhone}" class="px-4 py-2 bg-emerald-500 text-white rounded-xl text-sm hover:bg-emerald-600 transition-all hover:scale-105 flex items-center gap-2">
                                📞 {booking.hostPhone}
                              </a>
                              <a href="mailto:{booking.hostEmail}" class="px-4 py-2 bg-white/10 text-white rounded-xl text-sm hover:bg-white/20 transition-all hover:scale-105 flex items-center gap-2">
                                ✉️ {booking.hostEmail}
                              </a>
                            </div>
                            <div class="pt-3 border-t border-emerald-500/30 flex gap-3">
                              <button 
                                on:click={() => openExperienceModal(booking)}
                                class="flex-1 px-4 py-3 bg-gradient-to-r from-kerala-gold to-amber-500 text-white rounded-xl font-semibold hover:shadow-lg hover:shadow-kerala-gold/30 transition-all hover:scale-105 flex items-center justify-center gap-2"
                              >
                                ✍️ Share Experience & End Trip
                              </button>
                              <button 
                                on:click={() => markTripEnded(booking)}
                                class="px-4 py-3 bg-white/10 text-white/80 rounded-xl font-semibold hover:bg-white/20 transition-all hover:scale-105 flex items-center justify-center gap-2"
                              >
                                🏁 End Trip
                              </button>
                            </div>
                          </div>
                        {/if}
                      </div>
                    {/each}
                  </div>
                {/if}
              </div>
            </div>
          </div>

          <!-- Sidebar -->
          <div class="space-y-6">
            <!-- Experiences -->
            <div class="bg-white/10 backdrop-blur-md rounded-3xl border border-white/20 overflow-hidden animate-slide-up animation-delay-400">
              <div class="p-6 border-b border-white/10">
                <h2 class="text-xl font-bold text-white flex items-center gap-3">
                  <span class="w-10 h-10 bg-kerala-gold/20 rounded-xl flex items-center justify-center">⭐</span>
                  My Experiences
                </h2>
              </div>
              
              <div class="p-6">
                {#if experiences.length === 0}
                  <div class="text-center py-8">
                    <div class="text-4xl mb-3 animate-float">📝</div>
                    <p class="text-white/50 text-sm">Share your experiences after your trip!</p>
                  </div>
                {:else}
                  <div class="space-y-4">
                    {#each experiences as exp}
                      <div class="p-4 bg-white/5 rounded-xl border border-white/10 hover:bg-white/10 transition-all">
                        <div class="flex items-center gap-1 mb-2">
                          {#each Array(exp.rating) as _}
                            <span class="text-kerala-gold animate-pulse">⭐</span>
                          {/each}
                        </div>
                        <p class="text-white/80 text-sm mb-2 line-clamp-3">{exp.text}</p>
                        <p class="text-white/40 text-xs">📍 {exp.location} with {exp.hostName}</p>
                      </div>
                    {/each}
                  </div>
                {/if}
              </div>
            </div>
            
            <!-- Quick Actions -->
            <div class="bg-white/10 backdrop-blur-md rounded-3xl border border-white/20 overflow-hidden animate-slide-up animation-delay-600">
              <div class="p-6 border-b border-white/10">
                <h2 class="text-xl font-bold text-white flex items-center gap-3">
                  <span class="w-10 h-10 bg-kerala-gold/20 rounded-xl flex items-center justify-center">⚡</span>
                  Quick Actions
                </h2>
              </div>
              
              <div class="p-4 space-y-3">
                <a 
                  href="/plan-my-trip"
                  class="w-full p-4 bg-white/5 hover:bg-white/10 rounded-2xl border border-white/10 hover:border-kerala-gold/50 transition-all hover:scale-[1.02] text-left flex items-center gap-4 group"
                >
                  <div class="w-12 h-12 bg-gradient-to-br from-kerala-gold to-amber-600 rounded-xl flex items-center justify-center text-2xl group-hover:scale-110 transition-transform">
                    💬
                  </div>
                  <div>
                    <h3 class="font-semibold text-white">Plan My Trip</h3>
                    <p class="text-sm text-white/50">Access travel services</p>
                  </div>
                </a>
                
                <button on:click={() => window.location.href = '/matching'} class="w-full p-4 bg-white/5 hover:bg-white/10 rounded-2xl border border-white/10 hover:border-emerald-500/50 transition-all hover:scale-[1.02] text-left flex items-center gap-4 group">
                  <div class="w-12 h-12 bg-gradient-to-br from-emerald-500 to-teal-600 rounded-xl flex items-center justify-center text-2xl group-hover:scale-110 transition-transform">
                    🏠
                  </div>
                  <div>
                    <h3 class="font-semibold text-white">Browse Hosts</h3>
                    <p class="text-sm text-white/50">Find local experiences</p>
                  </div>
                </button>
                
                <button on:click={clearAllBookings} class="w-full p-4 bg-red-500/10 hover:bg-red-500/20 rounded-2xl border border-red-500/20 hover:border-red-500/50 transition-all hover:scale-[1.02] text-left flex items-center gap-4 group">
                  <div class="w-12 h-12 bg-gradient-to-br from-red-500 to-rose-600 rounded-xl flex items-center justify-center text-2xl group-hover:scale-110 transition-transform">
                    🗑️
                  </div>
                  <div>
                    <h3 class="font-semibold text-white">Clear All Bookings</h3>
                    <p class="text-sm text-white/50">Reset for testing</p>
                  </div>
                </button>
              </div>
            </div>
          </div>
        </div>
      </main>
    {/if}
  </div>

  <!-- Experience Modal -->
  {#if showExperienceModal && selectedBooking}
    <div class="fixed inset-0 z-50 flex items-center justify-center p-4">
      <div class="absolute inset-0 bg-black/80 backdrop-blur-md" on:click={closeExperienceModal}></div>
      
      <div class="relative bg-gradient-to-br from-gray-900 to-gray-800 rounded-3xl max-w-lg w-full shadow-2xl border border-white/20 animate-modal-pop overflow-hidden">
        <!-- Decorative elements -->
        <div class="absolute top-0 right-0 w-40 h-40 bg-kerala-gold/20 rounded-full filter blur-3xl"></div>
        <div class="absolute bottom-0 left-0 w-40 h-40 bg-emerald-500/20 rounded-full filter blur-3xl"></div>
        
        <div class="relative">
          <div class="bg-gradient-to-r from-kerala-gold to-amber-500 p-6">
            <button 
              on:click={closeExperienceModal}
              class="absolute top-4 right-4 w-10 h-10 bg-white/20 rounded-full flex items-center justify-center text-white hover:bg-white/30 transition-all hover:scale-110 hover:rotate-90"
            >
              ✕
            </button>
            <h2 class="text-2xl font-bold text-white">Share Your Experience ✨</h2>
            <p class="text-white/80">Tell us about your trip with {selectedBooking.hostName}</p>
          </div>
          
          <div class="p-6 space-y-6">
            <!-- Rating -->
            <div>
              <label class="block text-sm font-semibold text-white/80 mb-3">How was your experience?</label>
              <div class="flex gap-3 justify-center">
                {#each [1, 2, 3, 4, 5] as star}
                  <button 
                    on:click={() => experienceRating = star}
                    class="text-4xl transition-all duration-300 hover:scale-125 {star <= experienceRating ? 'text-kerala-gold scale-110' : 'text-white/30 grayscale'}"
                  >
                    ⭐
                  </button>
                {/each}
              </div>
            </div>
            
            <!-- Experience Text -->
            <div>
              <label class="block text-sm font-semibold text-white/80 mb-3">Share your story</label>
              <textarea 
                bind:value={experienceText}
                placeholder="What made this trip special? What activities did you enjoy?"
                class="w-full h-32 px-4 py-3 bg-white/10 border border-white/20 rounded-2xl text-white placeholder-white/40 focus:ring-2 focus:ring-kerala-gold focus:border-transparent resize-none transition-all"
              ></textarea>
            </div>
            
            <!-- Submit Button -->
            <button 
              on:click={submitExperience}
              disabled={!experienceText.trim()}
              class="w-full py-4 bg-gradient-to-r from-kerala-gold to-amber-500 text-white rounded-2xl font-bold text-lg hover:shadow-lg hover:shadow-kerala-gold/30 transition-all hover:scale-[1.02] disabled:opacity-50 disabled:hover:scale-100"
            >
              ✓ Submit Experience
            </button>
          </div>
        </div>
      </div>
    </div>
  {/if}
</div>

<style>
  @keyframes leafFall {
    0% { transform: translateY(-100px) rotate(0deg); opacity: 0; }
    10% { opacity: 1; }
    90% { opacity: 1; }
    100% { transform: translateY(100vh) rotate(360deg); opacity: 0; }
  }
  
  @keyframes float {
    0%, 100% { transform: translateY(0); }
    50% { transform: translateY(-20px); }
  }
  
  @keyframes slideUp {
    from { opacity: 0; transform: translateY(30px); }
    to { opacity: 1; transform: translateY(0); }
  }
  
  @keyframes confetti {
    0% { transform: translateY(-100vh) rotate(0deg); opacity: 1; }
    100% { transform: translateY(100vh) rotate(720deg); opacity: 0; }
  }
  
  @keyframes sway {
    0%, 100% { transform: rotate(-5deg) translateX(0); }
    50% { transform: rotate(5deg) translateX(10px); }
  }
  
  @keyframes gradient {
    0% { background-position: 0% 50%; }
    50% { background-position: 100% 50%; }
    100% { background-position: 0% 50%; }
  }
  
  @keyframes wordPop {
    0% { opacity: 0; transform: translateY(20px) scale(0.8); }
    50% { transform: translateY(-5px) scale(1.1); }
    100% { opacity: 1; transform: translateY(0) scale(1); }
  }
  
  @keyframes bounceIn {
    0% { opacity: 0; transform: scale(0); }
    50% { transform: scale(1.3); }
    100% { opacity: 1; transform: scale(1); }
  }
  
  @keyframes celebration {
    0%, 100% { transform: scale(1) rotate(0deg); }
    25% { transform: scale(1.1) rotate(-5deg); }
    50% { transform: scale(1) rotate(5deg); }
    75% { transform: scale(1.1) rotate(-3deg); }
  }
  
  @keyframes sparkle {
    0%, 100% { opacity: 0; transform: scale(0) rotate(0deg); }
    50% { opacity: 1; transform: scale(1) rotate(180deg); }
  }
  
  @keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
  }
  
  @keyframes pulseSlow {
    0%, 100% { opacity: 0.3; transform: scale(1); }
    50% { opacity: 0.5; transform: scale(1.1); }
  }
  
  @keyframes modalPop {
    0% { opacity: 0; transform: scale(0.8) translateY(20px); }
    100% { opacity: 1; transform: scale(1) translateY(0); }
  }
  
  .animate-leaf-fall { animation: leafFall 15s linear infinite; }
  .animate-float { animation: float 4s ease-in-out infinite; }
  .animate-slide-up { animation: slideUp 0.6s ease-out forwards; opacity: 0; }
  .animate-confetti { animation: confetti 3s ease-in-out forwards; }
  .animate-sway { animation: sway 4s ease-in-out infinite; }
  .animate-gradient { animation: gradient 3s ease infinite; }
  .animate-word-pop { animation: wordPop 0.5s ease-out forwards; opacity: 0; }
  .animate-bounce-in { animation: bounceIn 0.6s ease-out forwards; opacity: 0; }
  .animate-celebration { animation: celebration 2s ease-in-out infinite; }
  .animate-sparkle { animation: sparkle 2s ease-in-out infinite; }
  .animate-fade-in { animation: fadeIn 0.8s ease-out forwards; opacity: 0; }
  .animate-pulse-slow { animation: pulseSlow 4s ease-in-out infinite; }
  .animate-modal-pop { animation: modalPop 0.4s ease-out forwards; }
  .animate-spin-slow { animation: spin 3s linear infinite; }
  
  .animation-delay-200 { animation-delay: 200ms; }
  .animation-delay-400 { animation-delay: 400ms; }
  .animation-delay-500 { animation-delay: 500ms; }
  .animation-delay-600 { animation-delay: 600ms; }
  .animation-delay-800 { animation-delay: 800ms; }
  .animation-delay-1000 { animation-delay: 1000ms; }
  .animation-delay-1200 { animation-delay: 1200ms; }
  .animation-delay-1500 { animation-delay: 1500ms; }
  .animation-delay-2000 { animation-delay: 2000ms; }
  .animation-delay-4000 { animation-delay: 4000ms; }
  
  .line-clamp-3 {
    display: -webkit-box;
    -webkit-line-clamp: 3;
    -webkit-box-orient: vertical;
    overflow: hidden;
  }
</style>
