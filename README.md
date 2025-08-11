/ Mobile Navigation Toggle
document.addEventListener('DOMContentLoaded', function() {
    const hamburger = document.querySelector('.hamburger');
    const navMenu = document.querySelector('.nav-menu');
    
    if (hamburger && navMenu) {
        hamburger.addEventListener('click', function() {
            hamburger.classList.toggle('active');
            navMenu.classList.toggle('active');
        });

        // Close mobile menu when clicking on a nav link
        document.querySelectorAll('.nav-menu a').forEach(link => {
            link.addEventListener('click', () => {
                hamburger.classList.remove('active');
                navMenu.classList.remove('active');
            });
        });
    }

    // Smooth scrolling for anchor links
    document.querySelectorAll('a[href^="#"]').forEach(anchor => {
        anchor.addEventListener('click', function (e) {
            e.preventDefault();
            const target = document.querySelector(this.getAttribute('href'));
            if (target) {
                target.scrollIntoView({
                    behavior: 'smooth',
                    block: 'start'
                });
            }
        });
    });

    // Download button functionality
    const downloadButtons = document.querySelectorAll('.download-btn');
    downloadButtons.forEach(button => {
        button.addEventListener('click', function(e) {
            e.preventDefault();
            const platform = this.getAttribute('data-platform');
            handleDownload(platform);
        });
    });

    // Navbar background on scroll
    window.addEventListener('scroll', function() {
        const navbar = document.querySelector('.navbar');
        if (window.scrollY > 50) {
            navbar.style.background = 'rgba(255, 255, 255, 0.98)';
        } else {
            navbar.style.background = 'rgba(255, 255, 255, 0.95)';
        }
    });

    // Intersection Observer for animations
    const observerOptions = {
        threshold: 0.1,
        rootMargin: '0px 0px -50px 0px'
    };

    const observer = new IntersectionObserver(function(entries) {
        entries.forEach(entry => {
            if (entry.isIntersecting) {
                entry.target.style.opacity = '1';
                entry.target.style.transform = 'translateY(0)';
            }
        });
    }, observerOptions);

    // Observe elements for animation
    document.querySelectorAll('.feature-card, .download-card, .privacy-card, .contact-card').forEach(el => {
        el.style.opacity = '0';
        el.style.transform = 'translateY(20px)';
        el.style.transition = 'opacity 0.6s ease, transform 0.6s ease';
        observer.observe(el);
    });
});

// Download handler function
function handleDownload(platform) {
    // Show loading state
    showLoadingModal();
    
    // Simulate download preparation (in real app, this would be actual download logic)
    setTimeout(() => {
        hideLoadingModal();
        
        // Check if user has granted necessary permissions
        checkPermissions(platform).then(hasPermissions => {
            if (hasPermissions) {
                initiateDownload(platform);
            } else {
                showPermissionModal(platform);
            }
        });
    }, 1500);
}

// Permission checking function
async function checkPermissions(platform) {
    // This is a mock function - in a real app, you'd check actual system permissions
    if (platform === 'macos' || platform === 'windows') {
        // For desktop platforms, we need screen recording and accessibility permissions
        return await mockDesktopPermissionCheck();
    } else if (platform === 'android' || platform === 'ios') {
        // For mobile platforms, we need accessibility services
        return await mockMobilePermissionCheck();
    }
    return false;
}

async function mockDesktopPermissionCheck() {
    // Mock permission check - always returns false to show permission modal
    return false;
}

async function mockMobilePermissionCheck() {
    // Mock permission check - always returns false to show permission modal
    return false;
}

// Download initiation function
function initiateDownload(platform) {
    const downloadUrls = {
        macos: '/downloads/HyperActiveTech-macOS.dmg',
        windows: '/downloads/HyperActiveTech-Windows.exe',
        android: '/downloads/HyperActiveTech-Android.apk',
        ios: 'https://apps.apple.com/app/hyperactivetech'
    };

    const url = downloadUrls[platform];
    
    if (platform === 'ios') {
        // Redirect to App Store
        window.open(url, '_blank');
    } else {
        // Create temporary download link
        const link = document.createElement('a');
        link.href = url;
        link.download = '';
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
    }

    // Show success message
    showSuccessMessage(platform);
}

// Modal functions
function showLoadingModal() {
    const modal = createModal('loading', {
        title: 'Preparing Download',
        content: `
            <div class="loading-spinner"></div>
            <p>Checking system compatibility and preparing your download...</p>
        `
    });
    document.body.appendChild(modal);
}

function hideLoadingModal() {
    const modal = document.querySelector('.modal[data-type="loading"]');
    if (modal) {
        modal.remove();
    }
}

function showPermissionModal(platform) {
    const permissions = getRequiredPermissions(platform);
    const modal = createModal('permission', {
        title: 'Permissions Required',
        content: `
            <div class="permission-info">
                <p>HyperActive Tech needs the following permissions to function properly:</p>
                <ul class="permission-list">
                    ${permissions.map(perm => `
                        <li>
                            <i class="fas fa-${perm.icon}"></i>
                            <div>
                                <strong>${perm.name}</strong>
                                <p>${perm.description}</p>
                            </div>
                        </li>
                    `).join('')}
                </ul>
                <div class="permission-guarantee">
                    <i class="fas fa-shield-check"></i>
                    <p>All processing happens locally on your device. Your privacy is guaranteed.</p>
                </div>
            </div>
        `,
        buttons: [
            { text: 'Grant Permissions & Download', action: () => grantPermissionsAndDownload(platform), primary: true },
            { text: 'Cancel', action: () => closeModal('permission') }
        ]
    });
    document.body.appendChild(modal);
}

function getRequiredPermissions(platform) {
    const commonPermissions = [
        {
            name: 'Screen Recording',
            description: 'Required to analyze screen content and understand application state',
            icon: 'desktop'
        }
    ];

    if (platform === 'macos' || platform === 'windows') {
        return [
            ...commonPermissions,
            {
                name: 'Accessibility Control',
                description: 'Enables keyboard and mouse automation',
                icon: 'keyboard'
            },
            {
                name: 'Microphone (Optional)',
                description: 'For voice command input',
                icon: 'microphone'
            }
        ];
    } else {
        return [
            ...commonPermissions,
            {
                name: 'Accessibility Services',
                description: 'Enables touch and UI automation on your device',
                icon: 'mobile-alt'
            },
            {
                name: 'Microphone (Optional)',
                description: 'For voice command input',
                icon: 'microphone'
            }
        ];
    }
}

function grantPermissionsAndDownload(platform) {
    closeModal('permission');
    
    // Show permission instructions
    showPermissionInstructions(platform);
}

function showPermissionInstructions(platform) {
    const instructions = getPermissionInstructions(platform);
    const modal = createModal('instructions', {
        title: 'Grant Permissions',
        content: `
            <div class="instruction-content">
                <p>Follow these steps to grant the necessary permissions:</p>
                <ol class="instruction-list">
                    ${instructions.map(instruction => `<li>${instruction}</li>`).join('')}
                </ol>
                <div class="instruction-note">
                    <i class="fas fa-info-circle"></i>
                    <p>You can revoke these permissions at any time from your system settings.</p>
                </div>
            </div>
        `,
        buttons: [
            { text: 'Download Now', action: () => { closeModal('instructions'); initiateDownload(platform); }, primary: true },
            { text: 'Cancel', action: () => closeModal('instructions') }
        ]
    });
    document.body.appendChild(modal);
}

function getPermissionInstructions(platform) {
    const instructions = {
        macos: [
            'Open System Preferences → Security & Privacy',
            'Click on "Privacy" tab',
            'Select "Screen Recording" and check HyperActive Tech',
            'Select "Accessibility" and check HyperActive Tech',
            'Restart the application if prompted'
        ],
        windows: [
            'Open Windows Settings → Privacy & Security',
            'Select "Camera" and enable for HyperActive Tech',
            'Go to "Microphone" and enable if you want voice commands',
            'The app will request additional permissions when first launched'
        ],
        android: [
            'Go to Settings → Accessibility',
            'Find and enable HyperActive Tech',
            'Grant "Display over other apps" permission',
            'Allow microphone access if you want voice commands'
        ],
        ios: [
            'The app will request permissions when first launched',
            'Allow accessibility access in Settings → Accessibility → Touch',
            'Grant microphone permission for voice commands',
            'Note: iOS has stricter automation limitations'
        ]
    };

    return instructions[platform] || [];
}

function showSuccessMessage(platform) {
    const modal = createModal('success', {
        title: 'Download Started!',
        content: `
            <div class="success-content">
                <i class="fas fa-check-circle"></i>
                <p>Your download has started successfully!</p>
                <div class="next-steps">
                    <h4>Next Steps:</h4>
                    <ul>
                        <li>Install the application</li>
                        <li>Grant the required permissions</li>
                        <li>Start automating with voice commands!</li>
                    </ul>
                </div>
                <div class="support-info">
                    <p>Need help? Contact our <a href="#contact">support team</a></p>
                </div>
            </div>
        `,
        buttons: [
            { text: 'Got it!', action: () => closeModal('success'), primary: true }
        ]
    });
    document.body.appendChild(modal);
}

// Modal utility functions
function createModal(type, options) {
    const modal = document.createElement('div');
    modal.className = 'modal';
    modal.setAttribute('data-type', type);
    
    modal.innerHTML = `
        <div class="modal-overlay" onclick="closeModal('${type}')"></div>
        <div class="modal-content">
            <div class="modal-header">
                <h3>${options.title}</h3>
                <button class="modal-close" onclick="closeModal('${type}')">
                    <i class="fas fa-times"></i>
                </button>
            </div>
            <div class="modal-body">
                ${options.content}
            </div>
            ${options.buttons ? `
                <div class="modal-footer">
                    ${options.buttons.map(button => `
                        <button class="btn ${button.primary ? 'btn-primary' : 'btn-secondary'}" 
                                onclick="(${button.action.toString()})()">${button.text}</button>
                    `).join('')}
                </div>
            ` : ''}
        </div>
    `;

    // Add modal styles
    if (!document.querySelector('#modal-styles')) {
        const styles = document.createElement('style');
        styles.id = 'modal-styles';
        styles.textContent = `
            .modal {
                position: fixed;
                top: 0;
                left: 0;
                width: 100%;
                height: 100%;
                z-index: 10000;
                display: flex;
                align-items: center;
                justify-content: center;
            }

            .modal-overlay {
                position: absolute;
                top: 0;
                left: 0;
                width: 100%;
                height: 100%;
                background: rgba(0, 0, 0, 0.5);
                backdrop-filter: blur(5px);
            }

            .modal-content {
                background: white;
                border-radius: 16px;
                box-shadow: 0 25px 50px rgba(0, 0, 0, 0.25);
                max-width: 500px;
                width: 90%;
                max-height: 80vh;
                overflow-y: auto;
                position: relative;
                z-index: 1;
            }

            .modal-header {
                display: flex;
                justify-content: space-between;
                align-items: center;
                padding: 1.5rem;
                border-bottom: 1px solid #e2e8f0;
            }

            .modal-header h3 {
                margin: 0;
                color: #1a202c;
            }

            .modal-close {
                background: none;
                border: none;
                font-size: 1.25rem;
                color: #64748b;
                cursor: pointer;
                padding: 0.5rem;
                border-radius: 8px;
                transition: all 0.2s ease;
            }

            .modal-close:hover {
                background: #f1f5f9;
                color: #334155;
            }

            .modal-body {
                padding: 1.5rem;
            }

            .modal-footer {
                padding: 1.5rem;
                border-top: 1px solid #e2e8f0;
                display: flex;
                gap: 1rem;
                justify-content: flex-end;
            }

            .loading-spinner {
                width: 40px;
                height: 40px;
                border: 4px solid #e2e8f0;
                border-top: 4px solid #667eea;
                border-radius: 50%;
                animation: spin 1s linear infinite;
                margin: 0 auto 1rem;
            }

            @keyframes spin {
                0% { transform: rotate(0deg); }
                100% { transform: rotate(360deg); }
            }

            .permission-list, .instruction-list {
                margin: 1rem 0;
            }

            .permission-list li {
                display: flex;
                align-items: flex-start;
                gap: 1rem;
                margin-bottom: 1rem;
                padding: 1rem;
                background: #f8fafc;
                border-radius: 8px;
            }

            .permission-list li i {
                color: #667eea;
                font-size: 1.25rem;
                margin-top: 0.25rem;
            }

            .permission-guarantee, .instruction-note {
                display: flex;
                align-items: center;
                gap: 0.75rem;
                background: #ecfdf5;
                padding: 1rem;
                border-radius: 8px;
                margin-top: 1rem;
            }

            .permission-guarantee i, .instruction-note i {
                color: #059669;
            }

            .success-content {
                text-align: center;
            }

            .success-content i {
                font-size: 3rem;
                color: #059669;
                margin-bottom: 1rem;
            }

            .next-steps {
                background: #f8fafc;
                padding: 1.5rem;
                border-radius: 8px;
                margin: 1rem 0;
                text-align: left;
            }

            .next-steps h4 {
                margin-bottom: 0.75rem;
                color: #1a202c;
            }

            .next-steps ul {
                margin: 0;
                padding-left: 1.25rem;
            }

            .next-steps li {
                margin-bottom: 0.5rem;
                color: #64748b;
            }

            .support-info {
                margin-top: 1rem;
                font-size: 0.875rem;
                color: #64748b;
            }

            .support-info a {
                color: #667eea;
                text-decoration: none;
            }
        `;
        document.head.appendChild(styles);
    }

    return modal;
}

function closeModal(type) {
    const modal = document.querySelector(`.modal[data-type="${type}"]`);
    if (modal) {
        modal.remove();
    }
}

    // Contact form handling
    const contactForm = document.getElementById('contact-form');
    if (contactForm) {
        contactForm.addEventListener('submit', function(e) {
            const submitBtn = this.querySelector('button[type="submit"]');
            const originalText = submitBtn.innerHTML;
            
            // Show loading state
            submitBtn.innerHTML = '<i class="fas fa-spinner fa-spin"></i> Sending...';
            submitBtn.disabled = true;
            
            // Reset button after delay (form will redirect)
            setTimeout(() => {
                submitBtn.innerHTML = originalText;
                submitBtn.disabled = false;
            }, 2000);
        });
    }

    // Analytics and tracking
function trackDownload(platform) {
    // Google Analytics tracking
    if (typeof gtag !== 'undefined') {
        gtag('event', 'download', {
            'platform': platform,
            'version': '2.1.0'
        });
    }
    
    console.log(`Download initiated for platform: ${platform}`);
}

function trackPermissionGranted(platform, permission) {
    // Google Analytics tracking
    if (typeof gtag !== 'undefined') {
        gtag('event', 'permission_granted', {
            'platform': platform,
            'permission': permission
        });
    }
    
    console.log(`Permission granted: ${permission} for ${platform}`);
}
