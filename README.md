# learnpro

import time
from collections import deque
from threading import Lock

class SlidingWindowRateLimiter:
    """Sliding window rate limiter using deque"""
    
    def __init__(self, max_requests: int, window_size: int):
        self.max_requests = max_requests
        self.window_size = window_size
        self.requests = {}
        self.lock = Lock()
    
    def is_allowed(self, client_id: str) -> bool:
        """Check if request is allowed"""
        with self.lock:
            current_time = time.time()
            
            if client_id not in self.requests:
                self.requests[client_id] = deque()
            
            # Remove expired timestamps
            while (self.requests[client_id] and 
                   self.requests[client_id][0] <= current_time - self.window_size):
                self.requests[client_id].popleft()
            
            # Check if under limit
            if len(self.requests[client_id]) < self.max_requests:
                self.requests[client_id].append(current_time)
                return True
            return False

# Leaky Bucket Rate Limiter

import time
from threading import Lock

class LeakyBucketRateLimiter:
    """Leaky bucket rate limiter - smooths out bursts"""
    
    def __init__(self, capacity: int, leak_rate: float):
        """
        Args:
            capacity: Maximum number of requests in bucket
            leak_rate: Requests processed per second
        """
        self.capacity = capacity
        self.leak_rate = leak_rate
        self.buckets = {}
        self.lock = Lock()
    
    def is_allowed(self, client_id: str) -> bool:
        """Check if request is allowed"""
        with self.lock:
            current_time = time.time()
            
            if client_id not in self.buckets:
                self.buckets[client_id] = {
                    'water': 0,
                    'last_leak': current_time
                }
            
            bucket = self.buckets[client_id]
            
            # Calculate leaked water
            elapsed = current_time - bucket['last_leak']
            leaked = elapsed * self.leak_rate
            bucket['water'] = max(0, bucket['water'] - leaked)
            bucket['last_leak'] = current_time
            
            # Check if can add more water
            if bucket['water'] < self.capacity:
                bucket['water'] += 1
                return True
            return False

