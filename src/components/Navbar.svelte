<script>
  let isMenuOpen = false;
  let showModal = false;
  let modalType = 'signin';
  let userRole = ''; // 'host' or 'traveller'
  let isSubmitting = false;
  let submitMessage = '';
  let submitSuccess = false;
  let existingUsers = [];

  // Form data
  let formData = {
    fullName: '',
    email: '',
    password: '',
    phone: '',
    location: '',
    aboutYou: '',
    specialExperience: '',
    activities: []
  };

  // Separate APIs for Host and Traveller
  const HOST_API_URL = 'https://script.google.com/macros/s/AKfycbyMJOX6SNcCuExmI_98GlgFRBvHfSxAXS-7DwWgxcXx1ZCOBTex0Z1ax3oEYqKRZ-LiJQ/exec';
  const TRAVELLER_API_URL = 'https://script.google.com/macros/s/AKfycbx_kfO6Rx1bECp4aoJ8q8YdKQUipgdbd4g7zTzpJc3JC5uP3076cvfE6LasswA4D9R0Jg/exec';

  // Get the correct API URL based on user role
  function getApiUrl() {
    return userRole === 'host' ? HOST_API_URL : TRAVELLER_API_URL;
  }

  // Fetch existing users based on role
  async function fetchExistingUsers() {
    try {
      const response = await fetch(getApiUrl());
      if (response.ok) {
        existingUsers = await response.json();
      }
    } catch (error) {
      console.log('Could not fetch users');
      existingUsers = [];
    }
  }

  // Check if email already exists
  function checkEmailExists(email) {
    return existingUsers.some(user => 
      user.email && user.email.toString().toLowerCase().trim() === email.toLowerCase().trim()
    );
  }

  // Validate sign in credentials
  function validateSignIn(email, password) {
    const user = existingUsers.find(user => 
      user.email && user.email.toString().toLowerCase().trim() === email.toLowerCase().trim()
    );
    if (!user) {
      return { valid: false, message: 'Email not found. Please sign up first.' };
    }
    
    // Check password - try different possible field names
    const storedPassword = user.password || user.Password || user['password'] || '';
    const inputPassword = password.toString().trim();
    const storedPasswordTrimmed = storedPassword.toString().trim();
    
    if (storedPasswordTrimmed !== inputPassword) {
      return { valid: false, message: 'Incorrect password. Please try again.' };
    }
    return { valid: true, user };
  }

  function toggleMenu() {
    isMenuOpen = !isMenuOpen;
  }

  function openModal(type) {
    modalType = type;
    userRole = '';
    showModal = true;
    isMenuOpen = false;
    resetForm();
  }

  function closeModal() {
    showModal = false;
    userRole = '';
    resetForm();
  }

  async function selectRole(role) {
    userRole = role;
    existingUsers = [];
    await fetchExistingUsers();
  }

  function goBack() {
    userRole = '';
    existingUsers = [];
    resetForm();
  }

  function resetForm() {
    formData = {
      fullName: '',
      email: '',
      password: '',
      phone: '',
      location: '',
      aboutYou: '',
      specialExperience: '',
      activities: []
    };
    submitMessage = '';
    submitSuccess = false;
  }

  function toggleActivity(activityId) {
    if (formData.activities.includes(activityId)) {
      formData.activities = formData.activities.filter(a => a !== activityId);
    } else {
      formData.activities = [...formData.activities, activityId];
    }
  }

  async function handleSubmit(event) {
    event.preventDefault();
    isSubmitting = true;
    submitMessage = '';

    // Refresh user list before validation
    await fetchExistingUsers();

    // For Sign In - validate credentials
    if (modalType === 'signin') {
      const validation = validateSignIn(formData.email, formData.password);
      if (!validation.valid) {
        submitSuccess = false;
        submitMessage = validation.message;
        isSubmitting = false;
        return;
      }
      
      // Store user info in localStorage
      localStorage.setItem('keralaUser', JSON.stringify({
        name: validation.user.name,
        email: validation.user.email,
        role: userRole
      }));
      
      submitSuccess = true;
      submitMessage = `Welcome back, ${validation.user.name || 'User'}! Redirecting...`;
      
      setTimeout(() => {
        // Redirect to chat page
        window.location.href = '/chat';
      }, 1500);
      isSubmitting = false;
      return;
    }

    // For Sign Up - check if email already exists
    if (modalType === 'signup') {
      if (checkEmailExists(formData.email)) {
        submitSuccess = false;
        submitMessage = 'This email is already registered. Please sign in instead.';
        isSubmitting = false;
        
        // Auto switch to sign in after 2 seconds
        setTimeout(() => {
          modalType = 'signin';
          submitMessage = '';
        }, 2500);
        return;
      }
    }

    // Create payload based on role
    let payload;
    if (userRole === 'host') {
      // Host payload with all fields
      payload = {
        'name': formData.fullName,
        'place': formData.location || '',
        'about you': formData.aboutYou || '',
        'email': formData.email,
        'activities': formData.activities.join(', '),
        'experience': formData.specialExperience || '',
        'password': formData.password,
        'phone number': formData.phone || ''
      };
    } else {
      // Traveller payload - simpler with just name, email, password
      payload = {
        'name': formData.fullName,
        'email': formData.email,
        'password': formData.password
      };
    }

    try {
      await fetch(getApiUrl(), {
        method: 'POST',
        mode: 'no-cors',
        headers: {
          'Content-Type': 'text/plain'
        },
        body: JSON.stringify(payload)
      });

      submitSuccess = true;
      submitMessage = 'Account created successfully! Redirecting...';
      
      // Store user info in localStorage
      localStorage.setItem('keralaUser', JSON.stringify({
        name: formData.fullName,
        email: formData.email,
        role: userRole
      }));
      
      // Refresh user list
      await fetchExistingUsers();
      
      setTimeout(() => {
        // Redirect to chat page
        window.location.href = '/chat';
      }, 1500);

    } catch (error) {
      submitSuccess = false;
      submitMessage = 'Something went wrong. Please try again.';
    } finally {
      isSubmitting = false;
    }
  }
</script>

<nav class="fixed w-full z-50 bg-black/20 backdrop-blur-md">
  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <div class="flex items-center justify-between h-16">
      <a href="#" class="text-2xl font-bold text-white">
        Kerala<span class="text-kerala-gold">Tourism</span>
      </a>

      <ul class="hidden md:flex items-center space-x-8">
        <li><a href="#home" class="text-white hover:text-kerala-gold transition">Home</a></li>
        <li><a href="#destinations" class="text-white hover:text-kerala-gold transition">Destinations</a></li>
        <li><a href="#experiences" class="text-white hover:text-kerala-gold transition">Experiences</a></li>
        <li><a href="#about" class="text-white hover:text-kerala-gold transition">About</a></li>
        <li><a href="#contact" class="text-white hover:text-kerala-gold transition">Contact</a></li>
      </ul>

      <div class="hidden md:flex items-center space-x-4">
        <button on:click={() => openModal('signin')} class="px-4 py-2 text-white border border-white rounded-lg hover:bg-white hover:text-kerala-green transition">
          Sign In
        </button>
        <button on:click={() => openModal('signup')} class="px-4 py-2 bg-kerala-gold text-white rounded-lg hover:bg-yellow-600 transition">
          Sign Up
        </button>
      </div>

      <button on:click={toggleMenu} class="md:hidden text-white">
        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          {#if isMenuOpen}
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
          {:else}
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16" />
          {/if}
        </svg>
      </button>
    </div>
  </div>

  {#if isMenuOpen}
    <div class="md:hidden bg-black/90 backdrop-blur-md">
      <div class="px-4 py-4 space-y-3">
        <a href="#home" class="block text-white hover:text-kerala-gold">Home</a>
        <a href="#destinations" class="block text-white hover:text-kerala-gold">Destinations</a>
        <a href="#experiences" class="block text-white hover:text-kerala-gold">Experiences</a>
        <a href="#about" class="block text-white hover:text-kerala-gold">About</a>
        <a href="#contact" class="block text-white hover:text-kerala-gold">Contact</a>
        <div class="pt-4 space-y-2">
          <button on:click={() => openModal('signin')} class="w-full px-4 py-2 text-white border border-white rounded-lg">Sign In</button>
          <button on:click={() => openModal('signup')} class="w-full px-4 py-2 bg-kerala-gold text-white rounded-lg">Sign Up</button>
        </div>
      </div>
    </div>
  {/if}
</nav>

<!-- Auth Modal -->
{#if showModal}
  <div class="fixed inset-0 z-50 flex items-center justify-center bg-black/60 backdrop-blur-sm" on:click={closeModal}>
    <div class="bg-white rounded-2xl p-8 w-full max-w-md mx-4 shadow-2xl transform transition-all max-h-[90vh] overflow-y-auto" on:click|stopPropagation>
      
      <!-- Role Selection -->
      {#if !userRole}
        <div class="text-center">
          <div class="flex justify-end">
            <button on:click={closeModal} class="text-gray-500 hover:text-gray-700">
              <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
              </svg>
            </button>
          </div>
          <h2 class="text-2xl font-bold text-kerala-green mb-2">
            {modalType === 'signin' ? 'Welcome Back!' : 'Join Us'}
          </h2>
          <p class="text-gray-600 mb-8">Choose how you want to continue</p>
          
          <div class="grid grid-cols-2 gap-4">
            <button 
              on:click={() => selectRole('traveller')}
              class="group p-6 border-2 border-gray-200 rounded-xl hover:border-kerala-green hover:bg-kerala-green/5 transition-all duration-300"
            >
              <div class="text-4xl mb-3 group-hover:scale-110 transition-transform">🧳</div>
              <h3 class="font-semibold text-kerala-green">Traveller</h3>
              <p class="text-xs text-gray-500 mt-1">Explore & book experiences</p>
            </button>
            
            <button 
              on:click={() => selectRole('host')}
              class="group p-6 border-2 border-gray-200 rounded-xl hover:border-kerala-gold hover:bg-kerala-gold/5 transition-all duration-300"
            >
              <div class="text-4xl mb-3 group-hover:scale-110 transition-transform">🏠</div>
              <h3 class="font-semibold text-kerala-gold">Host</h3>
              <p class="text-xs text-gray-500 mt-1">List your property</p>
            </button>
          </div>
        </div>
      
      <!-- Auth Form -->
      {:else}
        <div>
          <div class="flex justify-between items-center mb-6">
            <button on:click={goBack} class="text-gray-500 hover:text-kerala-green transition flex items-center gap-1">
              <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
              </svg>
              Back
            </button>
            <button on:click={closeModal} class="text-gray-500 hover:text-gray-700">
              <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
              </svg>
            </button>
          </div>

          <div class="text-center mb-6">
            <span class="text-4xl">{userRole === 'host' ? '🏠' : '🧳'}</span>
            <h2 class="text-2xl font-bold mt-2 {userRole === 'host' ? 'text-kerala-gold' : 'text-kerala-green'}">
              {modalType === 'signin' ? 'Sign In' : 'Sign Up'} as {userRole === 'host' ? 'Host' : 'Traveller'}
            </h2>
          </div>

          <form class="space-y-4" on:submit={handleSubmit}>
            {#if modalType === 'signup'}
              <div>
                <label class="block text-sm font-medium text-gray-700 mb-1">Full Name</label>
                <input type="text" bind:value={formData.fullName} required class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 {userRole === 'host' ? 'focus:ring-kerala-gold' : 'focus:ring-kerala-green'} focus:border-transparent transition" placeholder="Enter your name">
              </div>
              
              {#if userRole === 'host'}
                <div>
                  <label class="block text-sm font-medium text-gray-700 mb-1">Location</label>
                  <select bind:value={formData.location} required class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-kerala-gold focus:border-transparent transition">
                    <option value="">Select your location</option>
                    <option value="Alleppey">Alleppey</option>
                    <option value="Munnar">Munnar</option>
                    <option value="Kovalam">Kovalam</option>
                    <option value="Wayanad">Wayanad</option>
                    <option value="Kochi">Kochi</option>
                    <option value="Thekkady">Thekkady</option>
                    <option value="Varkala">Varkala</option>
                    <option value="Kumarakom">Kumarakom</option>
                  </select>
                </div>
                
                <div>
                  <label class="block text-sm font-medium text-gray-700 mb-1">About You</label>
                  <textarea 
                    bind:value={formData.aboutYou}
                    rows="3" 
                    class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-kerala-gold focus:border-transparent transition resize-none" 
                    placeholder="Tell us about yourself, your interests, and what kind of person you are..."
                  ></textarea>
                </div>
                
                <div>
                  <label class="block text-sm font-medium text-gray-700 mb-2">Activities You Can Offer</label>
                  <div class="grid grid-cols-2 gap-2">
                    {#each [
                      { id: 'Backwater Tours', label: '🛶 Backwater Tours' },
                      { id: 'Cooking Classes', label: '🍳 Cooking Classes' },
                      { id: 'Ayurveda Sessions', label: '🌿 Ayurveda Sessions' },
                      { id: 'Trekking Guides', label: '🥾 Trekking Guides' },
                      { id: 'Fishing Trips', label: '🎣 Fishing Trips' },
                      { id: 'Cultural Shows', label: '💃 Cultural Shows' },
                      { id: 'Wildlife Safari', label: '🐘 Wildlife Safari' },
                      { id: 'Yoga Classes', label: '🧘 Yoga Classes' }
                    ] as activity}
                      <label class="flex items-center gap-2 p-2 border border-gray-200 rounded-lg cursor-pointer hover:bg-kerala-gold/5 hover:border-kerala-gold transition text-sm {formData.activities.includes(activity.id) ? 'bg-kerala-gold/10 border-kerala-gold' : ''}">
                        <input 
                          type="checkbox" 
                          checked={formData.activities.includes(activity.id)}
                          on:change={() => toggleActivity(activity.id)}
                          class="w-4 h-4 text-kerala-gold rounded focus:ring-kerala-gold"
                        >
                        <span>{activity.label}</span>
                      </label>
                    {/each}
                  </div>
                </div>
                
                <div>
                  <label class="block text-sm font-medium text-gray-700 mb-1">What Makes Your Experience Special?</label>
                  <textarea 
                    bind:value={formData.specialExperience}
                    rows="2" 
                    class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-kerala-gold focus:border-transparent transition resize-none" 
                    placeholder="Describe what unique experiences you can offer to travellers..."
                  ></textarea>
                </div>
              {/if}
            {/if}
            
            <div>
              <label class="block text-sm font-medium text-gray-700 mb-1">Email</label>
              <input type="email" bind:value={formData.email} required class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 {userRole === 'host' ? 'focus:ring-kerala-gold' : 'focus:ring-kerala-green'} focus:border-transparent transition" placeholder="Enter your email">
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-700 mb-1">Password</label>
              <input type="password" bind:value={formData.password} required class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 {userRole === 'host' ? 'focus:ring-kerala-gold' : 'focus:ring-kerala-green'} focus:border-transparent transition" placeholder="Enter your password">
            </div>
            {#if modalType === 'signup' && userRole === 'host'}
              <div>
                <label class="block text-sm font-medium text-gray-700 mb-1">Phone Number</label>
                <input type="tel" bind:value={formData.phone} required class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-kerala-gold focus:border-transparent transition" placeholder="+91 XXXXX XXXXX">
              </div>
            {/if}

            {#if submitMessage}
              <div class="p-3 rounded-lg text-center {submitSuccess ? 'bg-green-100 text-green-700' : 'bg-red-100 text-red-700'}">
                {submitMessage}
              </div>
            {/if}

            <button 
              type="submit" 
              disabled={isSubmitting}
              class="w-full py-3 {userRole === 'host' ? 'bg-kerala-gold hover:bg-yellow-600' : 'bg-kerala-green hover:bg-green-800'} text-white rounded-lg font-semibold transition transform hover:scale-105 disabled:opacity-50 disabled:cursor-not-allowed disabled:transform-none"
            >
              {#if isSubmitting}
                <span class="flex items-center justify-center gap-2">
                  <svg class="animate-spin h-5 w-5" viewBox="0 0 24 24">
                    <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none"></circle>
                    <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                  </svg>
                  Processing...
                </span>
              {:else}
                {modalType === 'signin' ? 'Sign In' : 'Create Account'}
              {/if}
            </button>
          </form>

          <p class="mt-4 text-center text-gray-600">
            {#if modalType === 'signin'}
              Don't have an account? <button on:click={() => modalType = 'signup'} class="{userRole === 'host' ? 'text-kerala-gold' : 'text-kerala-green'} font-semibold hover:underline">Sign Up</button>
            {:else}
              Already have an account? <button on:click={() => modalType = 'signin'} class="{userRole === 'host' ? 'text-kerala-gold' : 'text-kerala-green'} font-semibold hover:underline">Sign In</button>
            {/if}
          </p>
        </div>
      {/if}
    </div>
  </div>
{/if}
