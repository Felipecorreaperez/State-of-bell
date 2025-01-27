from qiskit import QuantumCircuit, QuantumRegister, ClassicalRegister
from qiskit_aer import Aer
from qiskit.visualization import plot_histogram
from qiskit.compiler import transpile
import numpy as np

class QuantumProtocols:
    def __init__(self):
        self.simulator = Aer.get_backend('qasm_simulator')
    
    def create_bell_state(self, bell_type='Phi+'):
        """
        Create one of the four Bell states:
        Phi+ = (|00⟩ + |11⟩)/√2  - Type 'Phi+'
        Phi- = (|00⟩ - |11⟩)/√2  - Type 'Phi-'
        Psi+ = (|01⟩ + |10⟩)/√2  - Type 'Psi+'
        Psi- = (|01⟩ - |10⟩)/√2  - Type 'Psi-'
        """
        # Create circuit with 2 qubits and 2 classical bits
        qc = QuantumCircuit(2, 2)
        
        # Create the Bell state based on type
        qc.h(0)  # Put first qubit in superposition
        qc.cx(0, 1)  # Entangle qubits
        
        if bell_type == 'Phi-':
            qc.z(1)  # Phase flip on second qubit
        elif bell_type == 'Psi+':
            qc.x(1)  # Bit flip on second qubit
        elif bell_type == 'Psi-':
            qc.x(1)  # Bit flip on second qubit
            qc.z(1)  # Phase flip on second qubit
            
        # Measure both qubits
        qc.measure_all()
        
        # Execute the circuit
        transpiled_qc = transpile(qc, self.simulator)
        job = self.simulator.run(transpiled_qc, shots=1000)
        result = job.result()
        counts = result.get_counts(qc)
        
        return counts, qc
    
    def demonstrate_entanglement(self):
        """
        Demonstrate all four Bell states
        """
        bell_states = ['Phi+', 'Phi-', 'Psi+', 'Psi-']
        results = {}
        
        for state in bell_states:
            counts, _ = self.create_bell_state(state)
            results[state] = counts
            
        return results
    
    def quantum_teleportation_with_bell_state(self):
        """
        Quantum teleportation using Bell state
        """
        qr = QuantumRegister(3, 'q')
        cr = ClassicalRegister(3, 'c')
        qc = QuantumCircuit(qr, cr)
        
        # Prepare state to teleport (|+⟩ state)
        qc.h(0)
        
        # Create Bell state between qubits 1 and 2
        qc.h(1)
        qc.cx(1, 2)
        
        # Show intermediate state by measuring
        qc.barrier()
        qc.measure_all()
        
        # Execute circuit
        transpiled_qc = transpile(qc, self.simulator)
        job = self.simulator.run(transpiled_qc, shots=1000)
        result = job.result()
        counts = result.get_counts(qc)
        
        return counts, qc
    
    def quantum_key_distribution(self, n_bits):
        """
        BB84 protocol with Bell state verification
        """
        alice_basis = np.random.randint(2, size=n_bits)
        bob_basis = np.random.randint(2, size=n_bits)
        alice_bits = np.random.randint(2, size=n_bits)
        
        shared_key = []
        bell_state_verifications = []
        
        for i in range(n_bits):
            # Create circuit for this bit
            qc = QuantumCircuit(2, 2)
            
            # Create Bell state first
            qc.h(0)
            qc.cx(0, 1)
            
            # Apply basis transformations
            if alice_basis[i]:
                qc.h(0)
            if bob_basis[i]:
                qc.h(1)
            
            qc.measure_all()
            
            # Execute
            transpiled_qc = transpile(qc, self.simulator)
            result = self.simulator.run(transpiled_qc, shots=1).result()
            measurement = list(result.get_counts().keys())[0]
            
            # Verify if bases match and record Bell state correlation
            if alice_basis[i] == bob_basis[i]:
                shared_key.append(measurement[1])
                bell_state_verifications.append(measurement)
        
        return shared_key, alice_basis, bob_basis, bell_state_verifications

def main():
    qp = QuantumProtocols()
    
    print("\n1. Bell States Demonstration:")
    bell_results = qp.demonstrate_entanglement()
    for state, counts in bell_results.items():
        print(f"\nBell State {state}:")
        print(f"Measurement results: {counts}")
        print(f"Expected state: {'(|00⟩ + |11⟩)/√2' if state == 'Phi+' else '(|00⟩ - |11⟩)/√2' if state == 'Phi-' else '(|01⟩ + |10⟩)/√2' if state == 'Psi+' else '(|01⟩ - |10⟩)/√2'}")
    
    print("\n2. Quantum Teleportation with Bell State:")
    teleport_counts, _ = qp.quantum_teleportation_with_bell_state()
    print(f"Teleportation results (with Bell state): {teleport_counts}")
    
    print("\n3. QKD with Bell State Verification:")
    n_bits = 20
    shared_key, alice_basis, bob_basis, bell_verifications = qp.quantum_key_distribution(n_bits)
    
    print(f"Number of bits attempted: {n_bits}")
    print(f"Alice's basis: {alice_basis}")
    print(f"Bob's basis: {bob_basis}")
    print(f"Bell state verifications: {bell_verifications}")
    print(f"Final shared key: {''.join(map(str, shared_key))}")
    print(f"Key length: {len(shared_key)} bits")

if __name__ == "__main__":
    main()
