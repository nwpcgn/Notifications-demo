<script>
  import { onMount } from 'svelte';
  
  let notifications = $state([]);
  let currentNotification = $state(null);
  let displayedText = $state('');
  let isTyping = $state(false);
  let isVisible = $state(false);
  
  const TYPING_SPEED = 50; // milliseconds per character
  const DISPLAY_DURATION = 4000; // how long to show the notification after typing (not used for confirm)
  const FADE_DURATION = 300; // fade out duration
  const CONFIRM_TYPING_SPEED = 30; // faster typing for confirm dialogs
  
  let typingTimeout;
  let displayTimeout;
  let fadeTimeout;
  
  // Process the notification queue
  const processQueue = () => {
    if (notifications.length > 0 && !currentNotification) {
      currentNotification = notifications[0];
      notifications = notifications.slice(1);
      startTypingAnimation();
    }
  };
  
  // Start the typing animation
  const startTypingAnimation = () => {
    if (!currentNotification) return;
    
    displayedText = '';
    isTyping = true;
    isVisible = true;
    
    const message = currentNotification.message;
    const speed = currentNotification.type === 'confirm' ? CONFIRM_TYPING_SPEED : TYPING_SPEED;
    let charIndex = 0;
    
    const typeNextChar = () => {
      if (charIndex < message.length) {
        displayedText += message[charIndex];
        charIndex++;
        typingTimeout = setTimeout(typeNextChar, speed);
      } else {
        isTyping = false;
        // Only start auto-dismiss timer for non-confirm notifications
        if (currentNotification.type !== 'confirm') {
          displayTimeout = setTimeout(removeCurrentNotification, DISPLAY_DURATION);
        }
      }
    };
    
    typeNextChar();
  };
  
  // Remove the current notification
  const removeCurrentNotification = () => {
    isVisible = false;
    
    fadeTimeout = setTimeout(() => {
      currentNotification = null;
      displayedText = '';
      // Process next notification in queue
      processQueue();
    }, FADE_DURATION);
  };
  
  // Handle confirm dialog response
  const handleConfirmResponse = (confirmed) => {
    if (currentNotification?.callback) {
      currentNotification.callback(confirmed);
    }
    removeCurrentNotification();
  };

  // Manual dismiss function (updated)
  const dismissNotification = () => {
    clearTimeout(typingTimeout);
    clearTimeout(displayTimeout);
    
    // For confirm dialogs, treat dismiss as cancel
    if (currentNotification?.type === 'confirm' && currentNotification?.callback) {
      currentNotification.callback(false);
    }
    
    removeCurrentNotification();
  };
  
  // Public method to add notifications
  export const addNotification = (notification) => {
    notifications = [...notifications, notification];
    processQueue();
  };
  
  // Watch for changes in notifications array
  $effect(() => {
    processQueue();
  });
  
  // Cleanup timeouts on component destroy
  onMount(() => {
    return () => {
      clearTimeout(typingTimeout);
      clearTimeout(displayTimeout);
      clearTimeout(fadeTimeout);
    };
  });
  
  // Get notification styles based on type
  const getNotificationStyles = (type) => {
    const baseStyles = "border-l-4 shadow-lg";
    
    switch (type) {
      case 'success':
        return `${baseStyles} bg-green-50 border-green-400 text-green-800`;
      case 'error':
        return `${baseStyles} bg-red-50 border-red-400 text-red-800`;
      case 'warning':
        return `${baseStyles} bg-yellow-50 border-yellow-400 text-yellow-800`;
      case 'confirm':
        return `${baseStyles} bg-purple-50 border-purple-400 text-purple-800`;
      case 'info':
      default:
        return `${baseStyles} bg-blue-50 border-blue-400 text-blue-800`;
    }
  };
  
  // Get icon based on notification type
  const getIcon = (type) => {
    switch (type) {
      case 'success':
        return '✓';
      case 'error':
        return '✕';
      case 'warning':
        return '⚠';
      case 'confirm':
        return '?';
      case 'info':
      default:
        return 'ℹ';
    }
  };
</script>

<!-- Notification Container -->
{#if currentNotification}
  <div 
    class="fixed top-4 right-4 z-50 max-w-sm w-full transition-all duration-300 ease-in-out transform"
    class:opacity-100={isVisible}
    class:opacity-0={!isVisible}
    class:translate-y-0={isVisible}
    class:-translate-y-2={!isVisible}
    role={currentNotification.type === 'confirm' ? 'dialog' : 'alert'}
    aria-live="polite"
    aria-atomic="true"
    aria-modal={currentNotification.type === 'confirm' ? 'true' : 'false'}
  >
    <div class={`rounded-lg p-4 ${getNotificationStyles(currentNotification.type)}`}>
      <div class="flex items-start">
        <!-- Icon -->
        <div class="flex-shrink-0 mr-3">
          <span class="text-lg font-bold" aria-hidden="true">
            {getIcon(currentNotification.type)}
          </span>
        </div>
        
        <!-- Message Content -->
        <div class="flex-1 min-w-0">
          <p class="text-sm font-medium leading-5">
            {displayedText}
            {#if isTyping}
              <span class="animate-pulse">|</span>
            {/if}
          </p>
        </div>
        
        <!-- Close Button (only for non-confirm notifications) -->
        {#if currentNotification.type !== 'confirm'}
          <div class="flex-shrink-0 ml-3">
            <button
              onclick={dismissNotification}
              class="inline-flex rounded-md p-1.5 hover:bg-black hover:bg-opacity-10 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-offset-transparent focus:ring-current transition-colors"
              aria-label="Dismiss notification"
            >
              <span class="text-lg leading-none" aria-hidden="true">×</span>
            </button>
          </div>
        {/if}
      </div>
      
      <!-- Confirm Dialog Buttons -->
      {#if currentNotification.type === 'confirm' && !isTyping}
        <div class="mt-4 flex space-x-3 justify-end">
          <button
            onclick={() => handleConfirmResponse(false)}
            class="px-3 py-2 text-sm font-medium text-gray-700 bg-white border border-gray-300 rounded-md hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-purple-500 transition-colors"
          >
            {currentNotification.cancelText || 'Cancel'}
          </button>
          <button
            onclick={() => handleConfirmResponse(true)}
            class="px-3 py-2 text-sm font-medium text-white bg-purple-600 border border-transparent rounded-md hover:bg-purple-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-purple-500 transition-colors"
          >
            {currentNotification.confirmText || 'Confirm'}
          </button>
        </div>
      {/if}
      
      <!-- Progress Bar (only for non-confirm notifications) -->
      {#if !isTyping && isVisible && currentNotification.type !== 'confirm'}
        <div class="mt-2 w-full bg-black bg-opacity-10 rounded-full h-1">
          <div 
            class="bg-current h-1 rounded-full transition-all ease-linear"
            style="width: 100%; animation: shrink {DISPLAY_DURATION}ms linear forwards;"
          ></div>
        </div>
      {/if}
    </div>
  </div>
{/if}

<!-- Queue Indicator -->
{#if notifications.length > 0}
  <div class="fixed top-4 left-4 z-40">
    <div class="bg-gray-800 text-white px-3 py-2 rounded-lg shadow-lg">
      <span class="text-sm font-medium">
        {notifications.length} more notification{notifications.length === 1 ? '' : 's'}
      </span>
    </div>
  </div>
{/if}

<style>
  @keyframes shrink {
    from {
      width: 100%;
    }
    to {
      width: 0%;
    }
  }
</style>
