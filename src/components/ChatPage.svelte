<script>
  import { onMount } from 'svelte';
  
  let messages = [];
  let inputMessage = '';
  let userName = '';
  let gridSize = 20; // Number of cells per row/column
  let cells = [];
  let isLoading = false;
  let backgroundImage = '';
  let chatHistory = [];
  let recommendedPlace = ''; // Store the recommended place
  let awaitingConfirmation = false; // Track if we're waiting for user confirmation
  
  // Groq AI API - Use environment variable
  // IMPORTANT: Set your API key in .env file as PUBLIC_GROQ_API_KEY
  const GROQ_API_KEY = import.meta.env.PUBLIC_GROQ_API_KEY || '';
  const GROQ_API_URL = 'https://api.groq.com/openai/v1/chat/completions';
  
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
  
  // Call Groq AI API
  async function getAIResponse(userMessage) {
    const systemPrompt = `You are an expert Kerala tourism assistant. Your job is to analyze the user's travel preferences and recommend ONE PERFECT DESTINATION that matches ALL their interests.

CRITICAL RULES:
1. When user describes their travel style/preferences, recommend ONLY ONE specific place/region
2. DO NOT list multiple places across Kerala
3. Focus deeply on that ONE place with comprehensive details
4. Match the place to their specific interests (adventure, peace, food, culture, etc.)

PLACE MATCHING GUIDE:

For "backwaters + local food + village life + peaceful + boating":
→ Recommend: **ALLEPPEY (Alappuzha)**

For "adventure + trekking + nature + wildlife":
→ Recommend: **WAYANAD**

For "tea gardens + scenic beauty + cool weather + romantic":
→ Recommend: **MUNNAR**

For "beaches + surfing + ayurveda + relaxation":
→ Recommend: **KOVALAM** or **VARKALA**

For "culture + heritage + history + art":
→ Recommend: **KOCHI (Fort Kochi)**

For "wildlife + jungle + spices + boat safari":
→ Recommend: **THEKKADY**

For "spiritual + temples + traditions":
→ Recommend: **THRISSUR** or **GURUVAYUR**

RESPONSE FORMAT FOR PERSONALIZED QUERIES:

🎯 **PERFECT MATCH FOR YOU: [PLACE NAME]**

📍 **About [Place Name]:**
• Location: [District, how to reach]
• Why it's perfect for you: [Match with their interests]

🌟 **What You'll Experience:**
• [Experience 1 matching their interest]
• [Experience 2 matching their interest]
• [Experience 3 matching their interest]
• [Experience 4 matching their interest]

🍽️ **Local Food to Try:**
• [Dish 1] - [Description]
• [Dish 2] - [Description]
• [Dish 3] - [Description]

🎭 **Local Culture & Traditions:**
• [Tradition/Art form 1]
• [Tradition/Art form 2]

🏠 **Where to Stay:**
• Budget: [Option]
• Mid-range: [Option]
• Luxury: [Option]

📅 **Ideal Duration:** [X days]

🌤️ **Best Time to Visit:** [Months]

💡 **Insider Tips:**
• [Tip 1]
• [Tip 2]
• [Tip 3]

DETAILED PLACE KNOWLEDGE:

**ALLEPPEY (Alappuzha)** - Perfect for: backwaters, houseboats, village life, local food, peaceful
• Experiences: Houseboat stay on Vembanad Lake, canoe rides through narrow canals, toddy shop visits, coir making villages, Marari beach, Ambalapuzha temple payasam
• Food: Karimeen (pearl spot fish), duck roast, tapioca with fish curry, toddy, Alleppey fish curry
• Culture: Nehru Trophy boat race, coir weaving, fishing communities
• Stay: Houseboats, Lake resorts, Homestays in Kuttanad

**WAYANAD** - Perfect for: adventure, trekking, nature, wildlife, tribal culture
• Experiences: Chembra Peak trek (heart-shaped lake), Edakkal Caves, Banasura Sagar Dam, Soochipara Falls, Wayanad Wildlife Sanctuary, bamboo rafting
• Food: Tribal cuisine, bamboo rice, wild honey, organic coffee
• Culture: Tribal settlements, Thirunelli temple, Pazhassi Raja history
• Stay: Treehouse resorts, Plantation stays, Eco lodges

**MUNNAR** - Perfect for: scenic beauty, tea gardens, cool weather, romantic, photography
• Experiences: Tea museum, Eravikulam National Park (Nilgiri Tahr), Top Station, Mattupetty Dam, Echo Point, flower gardens
• Food: Fresh tea, cardamom coffee, local Kerala meals
• Culture: Tea plantation heritage, Muthuvan tribes
• Stay: Tea estate bungalows, Resorts with valley views

**KOCHI (Fort Kochi)** - Perfect for: culture, heritage, history, art, cosmopolitan
• Experiences: Chinese fishing nets, Mattancherry Palace, Jewish Synagogue, Kathakali shows, art galleries, spice markets, sunset walks
• Food: Seafood, Malabar biryani, Kerala parotta, toddy shop food
• Culture: Portuguese-Dutch-British heritage, Kochi-Muziris Biennale, Kathakali, Kalaripayattu
• Stay: Heritage hotels, Boutique stays in Fort Kochi

**THEKKADY** - Perfect for: wildlife, jungle, spices, adventure, nature
• Experiences: Periyar boat safari, bamboo rafting, jungle patrol, spice plantation tours, elephant rides, tribal visits
• Food: Spice-infused dishes, cardamom tea, pepper chicken
• Culture: Mannan tribe, spice trade history
• Stay: Jungle lodges, Spice plantation stays

REMEMBER: Analyze user's description carefully and recommend ONLY ONE place that best matches ALL their interests. Give deep, comprehensive details about that ONE place.`;

    // Add to chat history
    chatHistory.push({ role: 'user', content: userMessage });
    
    // Keep only last 10 messages for context
    const recentHistory = chatHistory.slice(-10);
    
    try {
      const response = await fetch(GROQ_API_URL, {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${GROQ_API_KEY}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          model: 'llama-3.1-8b-instant',
          messages: [
            { role: 'system', content: systemPrompt },
            ...recentHistory
          ],
          temperature: 0.7,
          max_tokens: 1200
        })
      });
      
      if (!response.ok) {
        throw new Error('AI API error');
      }
      
      const data = await response.json();
      const aiMessage = data.choices[0]?.message?.content || 'I apologize, I could not process your request. Please try again.';
      
      // Add AI response to history
      chatHistory.push({ role: 'assistant', content: aiMessage });
      
      // Extract place name from response for matching
      const placeMatch = aiMessage.match(/\*\*(?:PERFECT MATCH FOR YOU:|RECOMMENDED:)?\s*([A-Z][A-Za-z\s()]+)\*\*/);
      if (placeMatch) {
        recommendedPlace = placeMatch[1].trim();
      }
      
      return aiMessage;
    } catch (error) {
      console.error('AI Error:', error);
      return "I'm having trouble connecting right now. 🌴\n\n📍 **Try asking about:**\n• Beaches (Kovalam, Varkala)\n• Hill stations (Munnar, Wayanad)\n• Backwaters (Alleppey, Kumarakom)\n• Wildlife (Thekkady, Wayanad)\n• Culture (Kochi, Thrissur)";
    }
  }
  
  // Navigate to matching page
  function goToMatching() {
    if (recommendedPlace) {
      localStorage.setItem('selectedPlace', recommendedPlace);
      window.location.href = '/matching';
    }
  }
  
  // Suggest other places
  async function suggestOtherPlaces() {
    isLoading = true;
    awaitingConfirmation = false;
    
    const otherPlacesQuery = `I'm not fully convinced about ${recommendedPlace}. Can you suggest a DIFFERENT place in Kerala that might also match my interests? Give me another option with the same detailed format.`;
    
    messages = [...messages, {
      type: 'user',
      text: "Show me other options 🔄",
      time: new Date()
    }];
    
    try {
      const aiResponse = await getAIResponse(otherPlacesQuery);
      
      messages = [...messages, {
        type: 'bot',
        text: aiResponse,
        time: new Date()
      }];
      
      if (recommendedPlace) {
        awaitingConfirmation = true;
        setTimeout(() => {
          messages = [...messages, {
            type: 'bot',
            text: `🏠 Would you like me to find local hosts in **${recommendedPlace}** who can give you an authentic experience?`,
            time: new Date(),
            showHostButton: true
          }];
        }, 1000);
      }
    } catch (error) {
      messages = [...messages, {
        type: 'bot',
        text: "Sorry, I couldn't fetch other options. Please try again! 🙏",
        time: new Date()
      }];
    } finally {
      isLoading = false;
    }
  }
  
  // Initialize grid cells
  onMount(async () => {
    // Select random background image
    backgroundImage = backgroundImages[Math.floor(Math.random() * backgroundImages.length)];
    
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
      text: `Welcome${userName ? ', ' + userName : ''}! 🌴 I'm your AI-powered Kerala travel assistant. Ask me anything about Kerala - places to visit, food to try, best times to travel, or help planning your itinerary!`,
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
    
    // Check if user is confirming to find hosts
    if (awaitingConfirmation && (userQuery.toLowerCase().includes('yes') || userQuery.toLowerCase().includes('find') || userQuery.toLowerCase().includes('host'))) {
      goToMatching();
      return;
    }
    
    // Add user message
    messages = [...messages, {
      type: 'user',
      text: userQuery,
      time: new Date()
    }];
    
    inputMessage = '';
    isLoading = true;
    awaitingConfirmation = false;
    
    try {
      // Get AI response
      const aiResponse = await getAIResponse(userQuery);
      
      // Add bot message
      messages = [...messages, {
        type: 'bot',
        text: aiResponse,
        time: new Date()
      }];
      
      // If a place was recommended, ask for confirmation
      if (recommendedPlace) {
        awaitingConfirmation = true;
        setTimeout(() => {
          messages = [...messages, {
            type: 'bot',
            text: `🏠 Would you like me to find local hosts in **${recommendedPlace}** who can give you an authentic experience? They can help with homestays, local tours, and hidden gems!`,
            time: new Date(),
            showHostButton: true
          }];
        }, 1000);
      }
    } catch (error) {
      messages = [...messages, {
        type: 'bot',
        text: "Sorry, I encountered an error. Please try again! 🙏",
        time: new Date()
      }];
    } finally {
      isLoading = false;
    }
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
              {#if message.showHostButton}
                <div class="flex gap-2 mt-3">
                  <button 
                    on:click={goToMatching}
                    class="flex-1 py-2 bg-gradient-to-r from-kerala-green to-kerala-gold text-white rounded-lg font-semibold hover:opacity-90 transition flex items-center justify-center gap-2"
                  >
                    🏠 Find Local Hosts
                  </button>
                  <button 
                    on:click={suggestOtherPlaces}
                    class="flex-1 py-2 bg-white border-2 border-kerala-green text-kerala-green rounded-lg font-semibold hover:bg-kerala-green/10 transition flex items-center justify-center gap-2"
                  >
                    🔄 Try Other Options
                  </button>
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
