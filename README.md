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
