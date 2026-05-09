// Shopping Cart Management
let cart = [];
let wishlist = [];

// Load data from localStorage on page load
document.addEventListener('DOMContentLoaded', () => {
    loadCartFromStorage();
    loadWishlistFromStorage();
    updateCartCount();
    setupEventListeners();
    observeElements();
});

// Setup Event Listeners
function setupEventListeners() {
    // Add to cart buttons
    document.querySelectorAll('.add-to-cart-btn').forEach(btn => {
        btn.addEventListener('click', (e) => addToCart(e));
    });

    // Wishlist buttons
    document.querySelectorAll('.wishlist-btn').forEach(btn => {
        btn.addEventListener('click', (e) => toggleWishlist(e));
    });

    // Newsletter subscription
    const newsletterForm = document.querySelector('.newsletter-form');
    if (newsletterForm) {
        newsletterForm.addEventListener('submit', (e) => handleNewsletterSubmit(e));
    }

    // View All Products button
    const viewAllBtn = document.querySelector('.view-all-btn');
    if (viewAllBtn) {
        viewAllBtn.addEventListener('click', () => {
            alert('Redirecting to full product catalog...');
        });
    }

    // Smooth scrolling for navigation links
    document.querySelectorAll('a[href^="#"]').forEach(link => {
        link.addEventListener('click', (e) => {
            const target = link.getAttribute('href');
            if (target !== '#' && document.querySelector(target)) {
                e.preventDefault();
                document.querySelector(target).scrollIntoView({ behavior: 'smooth' });
            }
        });
    });

    // Category cards
    document.querySelectorAll('.category-card').forEach(card => {
        card.addEventListener('click', () => {
            const categoryName = card.querySelector('h3').textContent;
            alert(`Exploring ${categoryName} collection...`);
        });
    });
}

// Add to Cart Function
function addToCart(e) {
    const productCard = e.target.closest('.product-card');
    const productName = productCard.querySelector('.product-info h3').textContent;
    const productPrice = productCard.querySelector('.current-price').textContent;
    const productImage = productCard.querySelector('.product-image img').src;

    const product = {
        id: Date.now(),
        name: productName,
        price: productPrice,
        image: productImage,
        quantity: 1
    };

    // Check if product already in cart
    const existingProduct = cart.find(item => item.name === productName);
    if (existingProduct) {
        existingProduct.quantity += 1;
    } else {
        cart.push(product);
    }

    saveCartToStorage();
    updateCartCount();
    showNotification(`${productName} added to cart!`);
    
    // Animate button
    e.target.textContent = 'Added!';
    e.target.style.background = '#4CAF50';
    setTimeout(() => {
        e.target.textContent = 'Add to Cart';
        e.target.style.background = 'var(--primary-color)';
    }, 2000);
}

// Toggle Wishlist Function
function toggleWishlist(e) {
    const productCard = e.currentTarget.closest('.product-card');
    const productName = productCard.querySelector('.product-info h3').textContent;
    const productImage = productCard.querySelector('.product-image img').src;

    const wishlistItem = wishlist.find(item => item.name === productName);

    if (wishlistItem) {
        wishlist = wishlist.filter(item => item.name !== productName);
        e.currentTarget.classList.remove('active');
        showNotification(`${productName} removed from wishlist`);
    } else {
        wishlist.push({ name: productName, image: productImage });
        e.currentTarget.classList.add('active');
        showNotification(`${productName} added to wishlist!`);
    }

    saveWishlistToStorage();
}

// Save Cart to LocalStorage
function saveCartToStorage() {
    localStorage.setItem('almassCart', JSON.stringify(cart));
}

// Load Cart from LocalStorage
function loadCartFromStorage() {
    const savedCart = localStorage.getItem('almassCart');
    if (savedCart) {
        cart = JSON.parse(savedCart);
    }
}

// Save Wishlist to LocalStorage
function saveWishlistToStorage() {
    localStorage.setItem('almassWishlist', JSON.stringify(wishlist));
}

// Load Wishlist from LocalStorage
function loadWishlistFromStorage() {
    const savedWishlist = localStorage.getItem('almassWishlist');
    if (savedWishlist) {
        wishlist = JSON.parse(savedWishlist);
        // Mark wishlist items as active
        updateWishlistUI();
    }
}

// Update Wishlist UI
function updateWishlistUI() {
    document.querySelectorAll('.wishlist-btn').forEach(btn => {
        const productCard = btn.closest('.product-card');
        const productName = productCard.querySelector('.product-info h3').textContent;
        
        if (wishlist.find(item => item.name === productName)) {
            btn.classList.add('active');
        } else {
            btn.classList.remove('active');
        }
    });
}

// Update Cart Count
function updateCartCount() {
    const cartCount = document.querySelector('.cart-count');
    const totalItems = cart.reduce((sum, item) => sum + item.quantity, 0);
    if (cartCount) {
        cartCount.textContent = totalItems;
    }
}

// Newsletter Subscription Handler
function handleNewsletterSubmit(e) {
    e.preventDefault();
    const email = e.target.querySelector('input[type="email"]').value;
    
    if (email) {
        showNotification(`Thank you! Check ${email} for exclusive offers.`);
        e.target.reset();
    }
}

// Show Notification
function showNotification(message) {
    const notification = document.createElement('div');
    notification.style.cssText = `
        position: fixed;
        top: 20px;
        right: 20px;
        background: #4CAF50;
        color: white;
        padding: 1rem 1.5rem;
        border-radius: 8px;
        z-index: 1000;
        animation: slideIn 0.3s ease;
        font-weight: 500;
        box-shadow: 0 4px 12px rgba(0,0,0,0.15);
    `;
    notification.textContent = message;
    document.body.appendChild(notification);

    setTimeout(() => {
        notification.style.animation = 'slideOut 0.3s ease';
        setTimeout(() => notification.remove(), 300);
    }, 3000);
}

// Intersection Observer for Scroll Animations
function observeElements() {
    const observerOptions = {
        threshold: 0.1,
        rootMargin: '0px 0px -50px 0px'
    };

    const observer = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
            if (entry.isIntersecting) {
                entry.target.style.animation = 'fadeInUp 0.6s ease';
                entry.target.style.opacity = '1';
            }
        });
    }, observerOptions);

    // Observe product cards, testimonials, and category cards
    document.querySelectorAll('.product-card, .testimonial-card, .category-card').forEach(el => {
        el.style.opacity = '0';
        observer.observe(el);
    });
}

// Cart Icon Click Handler
document.addEventListener('DOMContentLoaded', () => {
    const cartIcon = document.querySelector('a[href="#cart"]');
    if (cartIcon) {
        cartIcon.addEventListener('click', (e) => {
            e.preventDefault();
            if (cart.length === 0) {
                showNotification('Your cart is empty!');
            } else {
                const cartSummary = cart.map(item => `${item.name} x${item.quantity}`).join(', ');
                showNotification(`Cart contains: ${cartSummary}`);
            }
        });
    }

    // Wishlist Icon Click Handler
    const wishlistIcon = document.querySelector('a[href="#wishlist"]');
    if (wishlistIcon) {
        wishlistIcon.addEventListener('click', (e) => {
            e.preventDefault();
            if (wishlist.length === 0) {
                showNotification('Your wishlist is empty!');
            } else {
                const wishlistSummary = wishlist.map(item => item.name).join(', ');
                showNotification(`Wishlist: ${wishlistSummary}`);
            }
        });
    }
});

// Add CSS animations to document
const style = document.createElement('style');
style.textContent = `
    @keyframes slideIn {
        from {
            transform: translateX(400px);
            opacity: 0;
        }
        to {
            transform: translateX(0);
            opacity: 1;
        }
    }

    @keyframes slideOut {
        from {
            transform: translateX(0);
            opacity: 1;
        }
        to {
            transform: translateX(400px);
            opacity: 0;
        }
    }

    @keyframes fadeInUp {
        from {
            opacity: 0;
            transform: translateY(30px);
        }
        to {
            opacity: 1;
            transform: translateY(0);
        }
    }
`;
document.head.appendChild(style);

// Search functionality
document.addEventListener('DOMContentLoaded', () => {
    const searchIcon = document.querySelector('a[href="#search"]');
    if (searchIcon) {
        searchIcon.addEventListener('click', (e) => {
            e.preventDefault();
            const searchTerm = prompt('Search for products:');
            if (searchTerm) {
                showNotification(`Searching for "${searchTerm}"...`);
            }
        });
    }
});

// CTA Button Click Handler
document.addEventListener('DOMContentLoaded', () => {
    const ctaBtn = document.querySelector('.cta-btn');
    if (ctaBtn) {
        ctaBtn.addEventListener('click', () => {
            showNotification('Redirecting to products...');
            setTimeout(() => {
                document.querySelector('#products').scrollIntoView({ behavior: 'smooth' });
            }, 500);
        });
    }
});

// Log cart and wishlist on console for debugging
window.viewCart = () => console.log('Cart:', cart);
window.viewWishlist = () => console.log('Wishlist:', wishlist);
