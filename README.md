# PQCBenchGUI4

A desktop application for running, visualizing, and comparing post-quantum cryptography benchmarks.

## Post-Quantum Encryption Implementation

The application now includes native integration with real post-quantum cryptographic algorithms, starting with Kyber. This allows users to:

1. Generate Kyber key pairs
2. Encrypt messages using Kyber (with AES-GCM for data encryption)
3. Decrypt messages using Kyber

### Kyber Encryption

[Kyber](https://pq-crystals.org/kyber/) is a lattice-based key encapsulation mechanism (KEM) that was selected by NIST as a standard for post-quantum key establishment.

The implementation in this application:

- Uses the liboqs (Open Quantum Safe) library for Kyber operations
- Leverages OpenSSL for AES-GCM symmetric encryption
- Supports all three security levels: Kyber-512, Kyber-768, and Kyber-1024
- Includes a native Node.js addon with C++ bindings for performance

### Building the Native Addon

To build the Kyber native addon, follow these steps:

1. Install build prerequisites (Windows):
   - Visual Studio Build Tools with C++ workload
   - Python 3
2. Build the addon:

   ```bash
   npm run build-addon
   ```

3. Test the Kyber implementation:
   ```bash
   npm run test-kyber
   ```

### Usage in the Application

The encryption functionality is available in the "Run Encryption" section of the application:

1. Select a security parameter (512, 768, or 1024)
2. Generate a keypair using the "Generate Keys" button
3. Enter a plaintext message to encrypt
4. Click "Encrypt Message" to encrypt using the public key
5. Use the secret key to decrypt the ciphertext

### Technical Implementation

The encryption system is implemented with three main components:

1. **C++ Core (addons/kyber_encrypt.cpp)**

   - Implements Kyber KEM operations using liboqs
   - Handles hybrid encryption with AES-GCM from OpenSSL
   - Manages memory securely for cryptographic operations

2. **Node.js Addon (addons/kyber_node_addon.cpp)**

   - Provides JavaScript bindings to the C++ core
   - Handles Buffer conversions between Node.js and C++
   - Exposes three main functions: generateKeypair, encrypt, decrypt

3. **Electron Integration**
   - IPC handlers in the main process communicate with the native addon
   - Renderer process UI provides a user-friendly interface
   - Secure key handling following best practices

## Data Storage Implementation

The PQCBenchGUI4 application implements a flexible and robust data storage solution for managing benchmark results. This implementation follows Phase 4 of the project plan, providing a foundation for the visualization, comparison, and export functionality.

### Architecture

The data storage solution is built around a JSON file-based approach with the following components:

1. **BenchmarkStore** - A TypeScript class that encapsulates all data storage operations:

   - Handles read/write operations to a JSON file in Electron's user data directory
   - Provides methods for saving, retrieving, filtering, and deleting benchmark results
   - Supports complex queries for advanced data analysis and visualization

2. **IPC Interface** - A set of handlers in the main process that expose benchmark store operations to the renderer process:

   - Main process handles all file I/O operations
   - Renderer process communicates via IPC to request data
   - Type-safe operations with comprehensive error handling

3. **Utility Functions** - A utility module in the renderer process that wraps IPC calls:
   - Provides an easy-to-use API for components
   - Includes advanced queries for visualization and comparison features
   - Handles data transformation for charts and exports

### Data Schema

The benchmark results are stored with the following schema:

```typescript
interface BenchmarkResult {
	id: string; // Unique identifier
	algorithm: string; // Algorithm name (e.g., "kyber", "dilithium")
	securityParam: string; // Security parameter (e.g., "512", "768", "1024")
	metrics: { [key: string]: number }; // Flexible metrics object (keygen, encaps, sign, etc.)
	timestamp: string; // ISO timestamp of the benchmark run
	status: 'completed' | 'failed'; // Benchmark status
	error?: string; // Optional error message if failed
}
```

This flexible schema accommodates the heterogeneous nature of different cryptographic algorithms, allowing for:

- Different metric sets between algorithms (e.g., keygen/encaps/decaps for KEMs vs. keygen/sign/verify for signatures)
- Future extensions with new metrics without schema changes
- Storage of both successful and failed benchmark runs

### Usage Examples

**Running a Benchmark and Saving Results:**

```typescript
const params = { algorithm: 'kyber', securityParam: '512' };
const result = await benchmarkStoreUtils.runBenchmark(params);
```

**Querying Results for Visualization:**

```typescript
// Get all Kyber benchmarks
const kyberResults = await benchmarkStoreUtils.getBenchmarksByAlgorithm(
	'kyber'
);

// Get benchmarks from the last week
const lastWeek = new Date();
lastWeek.setDate(lastWeek.getDate() - 7);
const recentResults = await benchmarkStoreUtils.getBenchmarksByDateRange(
	lastWeek,
	new Date()
);

// Get average metrics for comparative analysis
const averageMetrics = await benchmarkStoreUtils.getAverageMetrics();
```

### Future Extensions

The data storage implementation has been designed with future requirements in mind:

1. **Visualization** - The flexible query methods support various visualization scenarios:

   - Time-series analysis of algorithm performance
   - Comparative bar charts of different algorithms
   - Statistical analysis of repeated benchmark runs

2. **Comparison** - Advanced queries enable sophisticated comparisons:

   - Side-by-side algorithm comparison with consistent metrics
   - Post-quantum vs. classical algorithm comparisons
   - Performance analysis across different security parameters

3. **Export** - The data model is ready for various export formats:
   - JSON export with full fidelity
   - CSV export with flattened metrics
   - Aggregated reports with statistical summaries

## Getting Started

// Development instructions
