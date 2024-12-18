<template>
    <Head title="Dashboard" />

    <AuthenticatedLayout>
        <template #header>
            <h2 class="text-xl font-semibold leading-tight text-white">
                Dashboard
            </h2>
        </template>

        <div class="py-12" v-if="isVisible">
            <div class="mx-auto max-w-7xl sm:px-6 lg:px-8">
                <div class="overflow-hidden bg-white shadow-sm sm:rounded-lg">
                    <div class="p-6 text-gray-900">
                        <!-- Conditionally render the login message -->
                        <p>You're logged in!</p>
                    </div>
                </div>
            </div>
        </div>

        <!-- Todo List Component -->
        <div class="mt-5">
            <TodoList />
        </div>
    </AuthenticatedLayout>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout.vue';
import TodoList from '@/Components/TodoList.vue';
import { Head } from '@inertiajs/vue3';

const isVisible = ref(true); // Flag to control visibility

onMounted(() => {
    // Set a timer to hide the text after 6 seconds
    setTimeout(() => {
        isVisible.value = false;
    }, 1000); // 6000 ms = 6 seconds
});
</script>
