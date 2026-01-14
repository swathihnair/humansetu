<script>
  import { onMount } from 'svelte';
  
  let messages = [];
  let inputMessage = '';
  let userName = '';
  let gridSize = 20; // Number of cells per row/column
  let cells = [];
  let isLoading = false;
  let backgroundImage = '';
  let placesData = [];
  
  // Places API URL
  const PLACES_API_URL = 'https://script.google.com/macros/s/AKfycbwJN5y85XTG_gAD4JdFx9ZTBTkThe0ixAMZBK10o1N0etluWkdCRXdZLdIUphrZvXj7Ew/exec';
  
  // Keywords for matching user queries
  const keywords = {
    districts: ['thiruvananthapuram', 'kollam', 'pathanamthitta', 'alappuzha', 'alleppey', 'kottayam', 'idukki', 'ernakulam', 'kochi', 'cochin', 'thrissur', 'palakkad', 'malappuram', 'kozhikode', 'calicut', 'wayanad', 'kannur', 'kasaragod', 'munnar'],
    types: ['beach', 'hill', 'mountain', 'waterfall', 'temple', 'church', 'mosque', 'backwater', 'wildlife', 'sanctuary', 'museum', 'fort', 'palace', 'lake', 'dam', 'tea', 'spice', 'ayurveda', 'trekking', 'adventure', 'nature', 'heritage', 'culture', 'pilgrimage', 'forest']
  };
  
  // Kerala artform and culture images
  const backgroundImages = [
    'https://images.unsplash.com/photo-1582510003544-4d00b7f74220?q=80&w=2070',
    'https://images.unsplash.com/photo-1593693411515-c20261bcad6e?q=80&w=2070',
    'https://images.unsplash.com/photo-1602216056096-3b40cc0c9944?q=80&w=2070',
    'https://images.unsplash.com/photo-1595815771614-ade9d652a65d?q=80&w=2070',
    'https://images.unsplash.com/photo-1609340757984-7e8f0b0c4c3e?q=80&w=2070',
    'https://images.unsplash.com/photo-1590050752117-238cb0fb12b1?q=80&w=2070',
    'https://images.unsplash.com/photo-1585409677983-0f6c41ca9c3b?q=80&w=2070',
    'https://images.unsplash.com/photo-1567157577867-05ccb1388e66?q=80&w=2070',
    'https://images.unsplash.com/photo-1514222134-b57cbb8ce073?q=80&w=2070',
    'https://images.unsplash.com/photo-1580281657702-257584239a55?q=80&w=2070'
  ];
  
  // Fetch places data from Google Sheet
  async function fetchPlacesData() {
    try {
      const response = await fetch(PLACES_API_URL);
      if (response.ok) {
        placesData = await response.json();
        console.log('Places loaded:', placesData.length);
      }
    } catch (error) {
      console.log('Could not fetch places data');
    }
  }
  
  // Extract keywords from user query
  function extractKeywords(query) {
    const lowerQuery = query.toLowerCase();
    const foundDistricts = [];
    const foundTypes = [];
    
    // Check for district names
    keywords.districts.forEach(district => {
      if (lowerQuery.includes(district)) {
        foundDistricts.push(district);
      }
    });
    
    // Check for place types
    keywords.types.forEach(type => {
      if (lowerQuery.includes(type)) {
        foundTypes.push(type);
      }
    });
    
    return { districts: foundDistricts, types: foundTypes };
  }
  
  // Search places based on keywords
  function searchPlaces(query) {
    const { districts, types } = extractKeywords(query);
    const lowerQuery = query.toLowerCase();
    let results = [];
    
    if (placesData.length === 0) return results;
    
    // Search by district
    if (districts.length > 0) {
      results = placesData.filter(place => {
        const placeDistrict = (place.district || '').toLowerCase();
        return districts.some(d => placeDistrict.includes(d));
      });
    }
    
    // Search by type
    if (types.length > 0) {
      const typeResults = placesData.filter(place => {
        const placeType = (place.type || '').toLowerCase();
        return types.some(t => placeType.includes(t));
      });
      
      if (results.length > 0) {
        // Intersection if we have both district and type
        results = results.filter(r => typeResults.some(t => t.viste === r.viste));
        if (results.length === 0) {
          results = typeResults; // Fallback to type results
        }
      } else {
        results = typeResults;
      }
    }
    
    // If no specific matches, do a general search
    if (results.length === 0) {
      results = placesData.filter(place => {
        const searchText = `${place.district || ''} ${place.viste || ''} ${place.type || ''}`.toLowerCase();
        return lowerQuery.split(' ').some(word => word.length > 2 && searchText.includes(word));
      });
    }
    
    // Limit results
    return results.slice(0, 6);
  }
  
  // Format places as response with images
  function formatPlacesResponse(places, query) {
    if (places.length === 0) {
      return {
        text: "🔍 I couldn't find specific places matching your query. Try mentioning:\n\n• A district (Munnar, Wayanad, Kochi, Alleppey)\n• A type (beach, hill, waterfall, temple, wildlife)\n\nOr tell me what kind of experience you're looking for!",
        places: []
      };
    }
    
    let text = `🌴 Based on your interests, here are some amazing places to visit:\n\n`;
    
    places.forEach((place, index) => {
      text += `${index + 1}. **${place.viste || 'Unknown'}**\n`;
      text += `   📍 ${place.district || 'Kerala'} • ${place.type || 'Attraction'}\n\n`;
    });
    
    text += `\nWould you like more details about any of these places?`;
    
    return { text, places };
  }
  
  // Initialize grid cells
  onMount(async () => {
    // Select random background image
    backgroundImage = backgroundImages[Math.floor(Math.random() * backgroundImages.length)];
    
    // Fetch places data
    await fetchPlacesData();
    
    // Check if user is logged in
    const storedUser = localStorage.getItem('keralaUser');
    if (storedUser) {
      const user = JSON.parse(storedUser);
      userName = user.name || 'Traveller';
    }
    
    // Create grid cells
    const totalCells = gridSize * gridSize;
    cells = Array(totalCells).fill(false);
    
    // Add welcome message
    messages = [{
      type: 'bot',
      text: `Welcome${userName ? ', ' + userName : ''}! 🌴 I'm your Kerala travel assistant. Tell me what kind of places you'd like to visit - beaches, hills, waterfalls, temples, wildlife, or any district you're interested in!`,
      time: new Date()
    }];
  });
  
  function revealCell(index) {
    cells[index] = true;
    cells = [...cells];
  }
  
  async function sendMessage() {
    if (!inputMessage.trim()) return;
    
    const userQuery = inputMessage;
    
    // Add user message
    messages = [...messages, {
      type: 'user',
      text: userQuery,
      time: new Date()
    }];
    
    inputMessage = '';
    isLoading = true;
    
    // Search for places
    const foundPlaces = searchPlaces(userQuery);
    const response = formatPlacesResponse(foundPlaces, userQuery);
    
    // Simulate typing delay
    setTimeout(() => {
      messages = [...messages, {
        type: 'bot',
        text: response.text,
        places: response.places,
        time: new Date()
      }];
      isLoading = false;
    }, 800);
  }
  
  function handleKeyPress(event) {
    if (event.key === 'Enter' && !event.shiftKey) {
      event.preventDefault();
      sendMessage();
    }
  }
  
  function goHome() {
    window.location.href = '/';
  }
  
  function logout() {
    localStorage.removeItem('keralaUser');
    window.location.href = '/';
  }
</script>

<div class="min-h-screen relative overflow-hidden">
  <!-- Background Image (Random Kerala art/culture) -->
  <div 
    class="absolute inset-0 bg-cover bg-center"
    style="background-image: url('{backgroundImage}');"
  ></div>
  
  <!-- Interactive Grid Overlay -->
  <div class="absolute inset-0 grid" style="grid-template-columns: repeat({gridSize}, 1fr);">
    {#each cells as revealed, i}
      <div 
        class="aspect-square transition-all duration-500 cursor-pointer {revealed ? 'bg-transparent' : 'bg-white'}"
        on:mouseenter={() => revealCell(i)}
        style="opacity: {revealed ? 0 : 0.95};"
      ></div>
    {/each}
  </div>
  
  <!-- Header -->
  <header class="relative z-10 bg-black/30 backdrop-blur-md">
    <div class="max-w-7xl mx-auto px-4 py-4 flex items-center justify-between">
      <button on:click={goHome} class="text-2xl font-bold text-white hover:text-kerala-gold transition">
        Kerala<span class="text-kerala-gold">Tourism</span>
      </button>
      <div class="flex items-center gap-4">
        {#if userName}
          <span class="text-white/80">Hello, {userName}</span>
        {/if}
        <button on:click={logout} class="px-4 py-2 text-white border border-white/50 rounded-lg hover:bg-white/10 transition">
          Logout
        </button>
      </div>
    </div>
  </header>
  
  <!-- Chat Container -->
  <div class="relative z-10 max-w-4xl mx-auto px-4 py-8 min-h-[calc(100vh-80px)] flex flex-col">
    <div class="text-center mb-6">
      <h1 class="text-3xl font-bold text-black drop-shadow-lg">Where to go in Kerala?</h1>
      <p class="text-white/80 mt-2 drop-shadow">Move your mouse to reveal the art of Kerala ✨</p>
    </div>
    
    <!-- Messages Area -->
    <div class="flex-1 bg-white/90 backdrop-blur-md rounded-2xl shadow-2xl p-6 mb-4 overflow-y-auto max-h-[60vh]">
      <div class="space-y-4">
        {#each messages as message}
          <div class="flex {message.type === 'user' ? 'justify-end' : 'justify-start'}">
            <div class="max-w-[85%] {message.type === 'user' ? 'bg-kerala-green text-white' : 'bg-gray-100 text-gray-800'} rounded-2xl px-4 py-3 {message.type === 'user' ? 'rounded-br-md' : 'rounded-bl-md'}">
              <p class="whitespace-pre-line">{message.text}</p>
              
              <!-- Place Cards with Images -->
              {#if message.places && message.places.length > 0}
                <div class="grid grid-cols-2 md:grid-cols-3 gap-3 mt-4">
                  {#each message.places as place}
                    <div class="bg-white rounded-xl overflow-hidden shadow-md hover:shadow-lg transition cursor-pointer group">
                      {#if place.link}
                        <div class="h-24 overflow-hidden">
                          <img 
                            src={place.link} 
                            alt={place.viste} 
                            class="w-full h-full object-cover group-hover:scale-110 transition duration-300"
                            on:error={(e) => e.target.src = 'https://images.unsplash.com/photo-1602216056096-3b40cc0c9944?q=80&w=400'}
                          />
                        </div>
                      {:else}
                        <div class="h-24 bg-gradient-to-br from-kerala-green to-kerala-gold flex items-center justify-center">
                          <span class="text-3xl">🌴</span>
                        </div>
                      {/if}
                      <div class="p-2">
                        <h4 class="font-semibold text-sm text-kerala-green truncate">{place.viste || 'Unknown'}</h4>
                        <p class="text-xs text-gray-500 truncate">{place.district || 'Kerala'}</p>
                        <span class="inline-block mt-1 px-2 py-0.5 bg-kerala-gold/20 text-kerala-gold text-xs rounded-full truncate">
                          {place.type || 'Attraction'}
                        </span>
                      </div>
                    </div>
                  {/each}
                </div>
              {/if}
              
              <span class="text-xs {message.type === 'user' ? 'text-white/70' : 'text-gray-500'} mt-2 block">
                {message.time.toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'})}
              </span>
            </div>
          </div>
        {/each}
        
        {#if isLoading}
          <div class="flex justify-start">
            <div class="bg-gray-100 rounded-2xl px-4 py-3 rounded-bl-md">
              <div class="flex gap-1">
                <span class="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style="animation-delay: 0ms"></span>
                <span class="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style="animation-delay: 150ms"></span>
                <span class="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style="animation-delay: 300ms"></span>
              </div>
            </div>
          </div>
        {/if}
      </div>
    </div>
    
    <!-- Input Area -->
    <div class="bg-white/90 backdrop-blur-md rounded-2xl shadow-2xl p-4">
      <div class="flex gap-3">
        <input 
          type="text" 
          bind:value={inputMessage}
          on:keypress={handleKeyPress}
          placeholder="Ask about places, activities, food, or anything about Kerala..."
          class="flex-1 px-4 py-3 border border-gray-200 rounded-xl focus:ring-2 focus:ring-kerala-green focus:border-transparent transition"
        />
        <button 
          on:click={sendMessage}
          disabled={!inputMessage.trim() || isLoading}
          class="px-6 py-3 bg-kerala-green text-white rounded-xl font-semibold hover:bg-green-800 transition disabled:opacity-50 disabled:cursor-not-allowed"
        >
          <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 19l9 2-9-18-9 18 9-2zm0 0v-8" />
          </svg>
        </button>
      </div>
      <div class="flex gap-2 mt-3 flex-wrap">
        {#each ['Beach', 'Hill station', 'Waterfall', 'Temple', 'Wildlife', 'Backwater', 'Munnar', 'Wayanad', 'Kochi'] as suggestion}
          <button 
            on:click={() => { inputMessage = `I want to visit ${suggestion}`; sendMessage(); }}
            class="px-3 py-1 bg-kerala-gold/20 text-kerala-gold rounded-full text-sm hover:bg-kerala-gold/30 transition"
          >
            {suggestion}
          </button>
        {/each}
      </div>
    </div>
  </div>
</div>

<style>
  .grid > div:hover ~ div {
    transition-delay: 50ms;
  }
</style>
