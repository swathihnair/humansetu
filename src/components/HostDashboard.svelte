<script>
  import { onMount } from 'svelte';
  
  let hostData = null;
  let userName = '';
  let userEmail = '';
  let bookingRequests = [];
  let isLoading = true;
  
  onMount(() => {
    // Get host info from localStorage
    const storedUser = localStorage.getItem('keralaUser');
    console.log('Stored user:', storedUser); // Debug
    
    if (storedUser) {
      const user = JSON.parse(storedUser);
      console.log('Parsed user:', user); // Debug
      console.log('User keys:', Object.keys(user)); // Debug
      
      userName = user.name || 'Host';
      userEmail = user.email || '';
      hostData = user.userData || user; // Try userData first, then fall back to user itself
      
      console.log('Host data:', hostData); // Debug
      console.log('Host data keys:', hostData ? Object.keys(hostData) : 'null'); // Debug - show all field names
      
      // Check if user is actually a host
      if (user.role !== 'host') {
        window.location.href = '/chat';
        return;
      }
    } else {
      // Not logged in, redirect to home
      window.location.href = '/';
      return;
    }
    
    isLoading = false;
    
    // Load booking requests from localStorage
    loadBookingRequests();
  });
  
  function loadBookingRequests() {
    const allRequests = JSON.parse(localStorage.getItem('bookingRequests') || '[]');
    console.log('All booking requests:', allRequests); // Debug
    console.log('Host email:', userEmail, 'Host name:', userName); // Debug
    
    // Filter requests for this host - check multiple fields
    bookingRequests = allRequests.filter(req => {
      const emailMatch = req.hostEmail && userEmail && req.hostEmail.toLowerCase() === userEmail.toLowerCase();
      const nameMatch = req.hostName && userName && req.hostName.toLowerCase() === userName.toLowerCase();
      return emailMatch || nameMatch;
    });
    
    console.log('Filtered requests for host:', bookingRequests); // Debug
  }
  
  // Helper to get value from hostData with multiple possible field names
  function getField(...fieldNames) {
    if (!hostData) return '';
    for (const name of fieldNames) {
      if (hostData[name] !== undefined && hostData[name] !== null && hostData[name] !== '') {
        return hostData[name];
      }
    }
    return '';
  }
  
  function getHostLocation() {
    const location = getField('Location', 'location', 'place', 'Place');
    return location || 'Not set';
  }
  
  function getHostAbout() {
    return getField('About You', 'about you', 'aboutYou', 'About', 'about');
  }
  
  function getHostActivities() {
    return getField('Activities', 'activities', 'Activity', 'activity');
  }
  
  function getHostPhone() {
    const phone = getField('Phone Number', 'phone number', 'Phone', 'phone', 'phoneNumber');
    return phone || 'Not set';
  }
  
  function getHostExperience() {
    return getField('Experience', 'experience', 'Special Experience', 'specialExperience');
  }
  
  function logout() {
    localStorage.removeItem('keralaUser');
    window.location.href = '/';
  }
  
  function goHome() {
    window.location.href = '/';
  }
  
  function acceptBooking(id) {
    // Update in bookingRequests localStorage
    const allRequests = JSON.parse(localStorage.getItem('bookingRequests') || '[]');
    const updatedRequests = allRequests.map(b => 
      b.id === id ? { ...b, status: 'confirmed' } : b
    );
    localStorage.setItem('bookingRequests', JSON.stringify(updatedRequests));
    
    // Update in travellerBookings localStorage
    const travellerBookings = JSON.parse(localStorage.getItem('travellerBookings') || '[]');
    const updatedTravellerBookings = travellerBookings.map(b => 
      b.id === id ? { ...b, status: 'confirmed' } : b
    );
    localStorage.setItem('travellerBookings', JSON.stringify(updatedTravellerBookings));
    
    // Update local state
    bookingRequests = bookingRequests.map(b => 
      b.id === id ? { ...b, status: 'confirmed' } : b
    );
  }
  
  function declineBooking(id) {
    // Update in bookingRequests localStorage
    const allRequests = JSON.parse(localStorage.getItem('bookingRequests') || '[]');
    const updatedRequests = allRequests.map(b => 
      b.id === id ? { ...b, status: 'declined' } : b
    );
    localStorage.setItem('bookingRequests', JSON.stringify(updatedRequests));
    
    // Update in travellerBookings localStorage
    const travellerBookings = JSON.parse(localStorage.getItem('travellerBookings') || '[]');
    const updatedTravellerBookings = travellerBookings.map(b => 
      b.id === id ? { ...b, status: 'declined' } : b
    );
    localStorage.setItem('travellerBookings', JSON.stringify(updatedTravellerBookings));
    
    // Update local state
    bookingRequests = bookingRequests.map(b => 
      b.id === id ? { ...b, status: 'declined' } : b
    );
  }
</script>

<div class="min-h-screen bg-gradient-to-br from-kerala-green/10 via-white to-kerala-gold/10">
  <!-- Header -->
  <header class="bg-white shadow-md">
    <div class="max-w-7xl mx-auto px-4 py-4 flex items-center justify-between">
      <button on:click={goHome} class="text-2xl font-bold text-kerala-green hover:text-kerala-gold transition">
        Kerala<span class="text-kerala-gold">Tourism</span>
      </button>
      <div class="flex items-center gap-4">
        <span class="text-gray-600">Welcome, <span class="font-semibold text-kerala-green">{userName}</span></span>
        <button on:click={logout} class="px-4 py-2 text-kerala-green border border-kerala-green rounded-lg hover:bg-kerala-green hover:text-white transition">
          Logout
        </button>
      </div>
    </div>
  </header>
  
  {#if isLoading}
    <div class="flex items-center justify-center h-[80vh]">
      <div class="text-center">
        <div class="w-16 h-16 border-4 border-kerala-green border-t-transparent rounded-full animate-spin mx-auto mb-4"></div>
        <p class="text-gray-600">Loading dashboard...</p>
      </div>
    </div>
  {:else}
    <main class="max-w-7xl mx-auto px-4 py-8">
      <!-- Welcome Banner -->
      <div class="bg-gradient-to-r from-kerala-green to-kerala-gold rounded-2xl p-8 text-white mb-8">
        <div class="flex items-center gap-4">
          <div class="w-20 h-20 bg-white/20 rounded-full flex items-center justify-center text-4xl">
            🏠
          </div>
          <div>
            <h1 class="text-3xl font-bold">Host Dashboard</h1>
            <p class="text-white/80">Manage your listings and connect with travellers</p>
          </div>
        </div>
      </div>
      
      <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
        <!-- Profile Card -->
        <div class="lg:col-span-1">
          <div class="bg-white rounded-2xl shadow-lg p-6">
            <h2 class="text-xl font-bold text-kerala-green mb-4 flex items-center gap-2">
              <span>👤</span> Your Profile
            </h2>
            
            <div class="space-y-4">
              <div>
                <label class="text-sm text-gray-500">Name</label>
                <p class="font-semibold text-gray-800">{userName}</p>
              </div>
              
              <div>
                <label class="text-sm text-gray-500">Email</label>
                <p class="font-semibold text-gray-800">{userEmail}</p>
              </div>
              
              <div>
                <label class="text-sm text-gray-500">Location</label>
                <p class="font-semibold text-gray-800">📍 {getHostLocation()}</p>
              </div>
              
              <div>
                <label class="text-sm text-gray-500">Phone</label>
                <p class="font-semibold text-gray-800">📞 {getHostPhone()}</p>
              </div>
              
              <div>
                <label class="text-sm text-gray-500">About You</label>
                <p class="text-gray-700 text-sm">{getHostAbout() || 'Not set'}</p>
              </div>
              
              {#if getHostActivities()}
                <div>
                  <label class="text-sm text-gray-500">Activities You Offer</label>
                  <div class="flex flex-wrap gap-2 mt-1">
                    {#each getHostActivities().split(',') as activity}
                      <span class="px-3 py-1 bg-kerala-gold/20 text-kerala-gold text-xs rounded-full">
                        {activity.trim()}
                      </span>
                    {/each}
                  </div>
                </div>
              {/if}
            </div>
            
            <button class="w-full mt-6 py-2 border border-kerala-green text-kerala-green rounded-lg hover:bg-kerala-green hover:text-white transition">
              Edit Profile
            </button>
          </div>
        </div>
        
        <!-- Main Content -->
        <div class="lg:col-span-2 space-y-8">
          <!-- Stats -->
          <div class="grid grid-cols-3 gap-4">
            <div class="bg-white rounded-xl shadow-lg p-6 text-center">
              <div class="text-3xl font-bold text-kerala-green">{bookingRequests.filter(b => b.status === 'pending').length}</div>
              <div class="text-gray-500 text-sm">Pending Requests</div>
            </div>
            <div class="bg-white rounded-xl shadow-lg p-6 text-center">
              <div class="text-3xl font-bold text-kerala-gold">{bookingRequests.filter(b => b.status === 'confirmed').length}</div>
              <div class="text-gray-500 text-sm">Confirmed</div>
            </div>
            <div class="bg-white rounded-xl shadow-lg p-6 text-center">
              <div class="text-3xl font-bold text-gray-400">{bookingRequests.length}</div>
              <div class="text-gray-500 text-sm">Total Bookings</div>
            </div>
          </div>
          
          <!-- Booking Requests -->
          <div class="bg-white rounded-2xl shadow-lg p-6">
            <h2 class="text-xl font-bold text-kerala-green mb-4 flex items-center gap-2">
              <span>📋</span> Booking Requests
            </h2>
            
            {#if bookingRequests.length === 0}
              <div class="text-center py-12 text-gray-500">
                <div class="text-4xl mb-2">📭</div>
                <p>No booking requests yet</p>
                <p class="text-sm">When travellers book your activities, they'll appear here</p>
              </div>
            {:else}
              <div class="space-y-4">
                {#each bookingRequests as booking}
                  <div class="border border-gray-200 rounded-xl p-4 hover:border-kerala-green/50 transition">
                    <div class="flex items-center justify-between">
                      <div>
                        <h3 class="font-semibold text-gray-800">{booking.traveller}</h3>
                        <p class="text-sm text-gray-500">{booking.activities || 'General Visit'} • {booking.place || booking.location}</p>
                        <p class="text-xs text-gray-400 mt-1">Requested on {new Date(booking.createdAt || booking.date).toLocaleDateString()}</p>
                      </div>
                      <div class="flex items-center gap-2">
                        {#if booking.status === 'pending'}
                          <button 
                            on:click={() => acceptBooking(booking.id)}
                            class="px-3 py-1 bg-kerala-green text-white text-sm rounded-lg hover:bg-green-700 transition"
                          >
                            Accept
                          </button>
                          <button 
                            on:click={() => declineBooking(booking.id)}
                            class="px-3 py-1 bg-red-500 text-white text-sm rounded-lg hover:bg-red-600 transition"
                          >
                            Decline
                          </button>
                        {:else if booking.status === 'confirmed'}
                          <span class="px-3 py-1 bg-green-100 text-green-700 text-sm rounded-full">✓ Confirmed</span>
                        {:else}
                          <span class="px-3 py-1 bg-red-100 text-red-700 text-sm rounded-full">✗ Declined</span>
                        {/if}
                      </div>
                    </div>
                  </div>
                {/each}
              </div>
            {/if}
          </div>
          
          <!-- Quick Actions -->
          <div class="bg-white rounded-2xl shadow-lg p-6">
            <h2 class="text-xl font-bold text-kerala-green mb-4 flex items-center gap-2">
              <span>⚡</span> Quick Actions
            </h2>
            <div class="grid grid-cols-2 gap-4">
              <button class="p-4 border border-gray-200 rounded-xl hover:border-kerala-green hover:bg-kerala-green/5 transition text-left">
                <div class="text-2xl mb-2">📸</div>
                <h3 class="font-semibold text-gray-800">Add Photos</h3>
                <p class="text-sm text-gray-500">Showcase your place</p>
              </button>
              <button class="p-4 border border-gray-200 rounded-xl hover:border-kerala-gold hover:bg-kerala-gold/5 transition text-left">
                <div class="text-2xl mb-2">🎯</div>
                <h3 class="font-semibold text-gray-800">Add Activity</h3>
                <p class="text-sm text-gray-500">Offer new experiences</p>
              </button>
              <button class="p-4 border border-gray-200 rounded-xl hover:border-kerala-green hover:bg-kerala-green/5 transition text-left">
                <div class="text-2xl mb-2">💬</div>
                <h3 class="font-semibold text-gray-800">Messages</h3>
                <p class="text-sm text-gray-500">Chat with travellers</p>
              </button>
              <button class="p-4 border border-gray-200 rounded-xl hover:border-kerala-gold hover:bg-kerala-gold/5 transition text-left">
                <div class="text-2xl mb-2">📊</div>
                <h3 class="font-semibold text-gray-800">Analytics</h3>
                <p class="text-sm text-gray-500">View your stats</p>
              </button>
            </div>
          </div>
        </div>
      </div>
    </main>
  {/if}
</div>
