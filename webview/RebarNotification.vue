<template>
    <div :class="notificationPositionClass">
        <transition-group :name="transitionNotificationName" tag="div">
            <div v-for="(notification, index) in reversedNotifications" :key="notification.id">
                <RebarNotify :type="notification.type" :message="notification.message" />
            </div>
        </transition-group>
    </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import { RebarNotification } from '../shared/interface.js';
import { NotifyEvents } from '../shared/events.js';
import { useEvents } from '../../../../webview/composables/useEvents.js';
import { useAudio } from '../../../../webview/composables/useAudio.js';
import { ASCNotifications } from '../shared/config.js';
import RebarNotify from './components/RebarNotify.vue';

const audio = useAudio();
const events = useEvents();
type RebarNotifyProps = Pick<RebarNotification, 'type' | 'message'>;

defineProps<RebarNotifyProps>();

let debugMode = true; // Set to true for debugging

interface VueNotification extends RebarNotification {
    id?: number;
    progress?: number;
    startTime?: number;
    elapsedSeconds?: number;
}

const notifications = ref<VueNotification[]>([]);

let timer: NodeJS.Timeout | null = null;

const reversedNotifications = computed(() => {
    return notifications.value.slice().reverse();
});

const notificationPositionClass = computed(() => {
    const positions: { [key: string]: string } = {
        'top-right': 'fixed top-4 right-4',
        'bottom-right': 'fixed bottom-4 right-4',
        'right-center': 'fixed top-1/2 right-4 transform -translate-y-1/2',
        'top-left': 'fixed top-4 left-4',
        'bottom-left': 'fixed bottom-4 left-4',
        'left-center': 'fixed top-1/2 left-4 transform -translate-y-1/2',
    };
    return `${positions[ASCNotifications.position]} z-50 space-x-4`;
});

const transitionNotificationName = computed(() => {
    if (ASCNotifications.position.includes('left')) {
        return 'label-slide-left';
    } else if (ASCNotifications.position.includes('right')) {
        return 'label-slide-right';
    }
    return '';
});

const addNotification = (notification: VueNotification) => {
    notification.progress = 0;
    notification.id = generateUniqueId();
    notification.startTime = Date.now();

    notifications.value.push(notification);

    if (!timer) {
        timer = setInterval(updateProgress, 10);
    }
};

const removeNotification = (index: number) => {
    notifications.value.splice(index, 1);

    if (notifications.value.length === 0 && timer) {
        clearInterval(timer);
        timer = null;
    }
};

function generateUniqueId() {
    return Date.now();
}

const updateProgress = () => {
    const currentTime = Date.now();

    for (let i = notifications.value.length - 1; i >= 0; i--) {
        const notification = notifications.value[i];
        if (notification.progress < 100) {
            const elapsedTime = currentTime - notification.startTime;
            notification.elapsedSeconds = Math.floor(elapsedTime / 1000);
            if (elapsedTime < notification.duration) {
                notification.progress = (elapsedTime / notification.duration) * 100;
            } else {
                notification.progress = 100;
                removeNotification(i);
            }
        }
    }
};

const addDebugNotification = () => {
    if (debugMode) {
        const debugNotification: VueNotification = {
            type: 'SUCCESS',
            message: 'You have sent 420€ to Tommy!',
            duration: 60000, // 1 hour in milliseconds
        };
        const debugNotification2: VueNotification = {
            type: 'WARNING',
            message: 'You have sent 420€ to Tommy!',
            duration: 60000, // 1 hour in milliseconds
        };
        const debugNotification3: VueNotification = {
            type: 'ERROR',
            message: 'You have sent 420€ to Tommy!',
            duration: 60000, // 1 hour in milliseconds
        };
        const debugNotification4: VueNotification = {
            type: 'INFO',
            message: 'You have sent 420€ to Tommy!',
            duration: 60000, // 1 hour in milliseconds
        };
        addNotification(debugNotification);
        addNotification(debugNotification2);
        addNotification(debugNotification3);
        addNotification(debugNotification4);
    }
};

const init = () => {
    events.on(NotifyEvents.toWebview.CREATE_REBAR_NOTIFICATION, (notification: VueNotification) => {
        if (notification.oggFile) {
            audio.play(`/sounds/${notification.oggFile}.ogg`);
        }
        addNotification(notification);
    });
};

onMounted(() => {
    addDebugNotification();
    init();
});
</script>

<style scoped>
/* Left to Right Transition */
.label-slide-left-enter-active,
.label-slide-left-leave-active {
    transition:
        transform 0.5s ease-out,
        opacity 0.5s ease-out;
}

.label-slide-left-enter-from {
    transform: translateX(-100%); /* Slide in from the left */
    opacity: 0;
}

.label-slide-left-enter-to {
    transform: translateX(0);
    opacity: 1;
}

.label-slide-left-leave-from {
    transform: translateX(0); /* Start from the normal position */
    opacity: 1;
}

.label-slide-left-leave-to {
    transform: translateX(-100%); /* Slide out to the left */
    opacity: 0;
}

/* Right to Left Transition */
.label-slide-right-enter-active,
.label-slide-right-leave-active {
    transition:
        transform 0.5s ease-out,
        opacity 0.5s ease-out;
}

.label-slide-right-enter-from {
    transform: translateX(100%); /* Slide in from the right */
    opacity: 0;
}

.label-slide-right-enter-to {
    transform: translateX(0);
    opacity: 1;
}

.label-slide-right-leave-from {
    transform: translateX(0); /* Start from the normal position */
    opacity: 1;
}

.label-slide-right-leave-to {
    transform: translateX(100%); /* Slide out to the right */
    opacity: 0;
}
</style>
