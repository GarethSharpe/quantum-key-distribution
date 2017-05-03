"""
QuantumKeyDistribution.py: Utilizes the IBM Q5 to demonstrate Quantum Key Distribution.
"""

__author__ = "Gareth Sharpe"
__date__ = "2017-3-12"
__version__ = "1.0.1"
__maintainer__ = "Gareth Sharpe"
__email__ = "shar1370@mylaurier.ca"
__status__ = "Production"

# Imports
from IBMQuantumExperience import IBMQuantumExperience
from Utilities import text_from_bits, text_to_bits, random_bits, encrypt, decrypt

#---------- SETUP ----------#

# Configure API arguments
# Use your own token found in your account settings
token = 'ae62fe37579104f45f6d8f3b02a86b7f3de4a3ec864980c041268859060d90c30e3e30ff695536567f6a52c991d553dc04e5625cf4529bead5946a059525efa7'
config = {"url": 'https://quantumexperience.ng.bluemix.net/api'}

# Configure API
# Config argument defaults to above
api = IBMQuantumExperience.IBMQuantumExperience(token)

# Configure experiment arguments
qasm = 'OPENQASM 2.0;\n\ninclude "qelib1.inc";\nqreg q[8];\ncreg c[8];'
# Change to real if real topology is desired
device = 'simulator'
shots = 100
name = 'Quantum Key Distribution'
timeout = 60

print('*----- CONSOLE OUTPUT -----*')
# Alice wants to send a msg M (n bit string)
M = 'Q'
bin_msg = text_to_bits(M)
n = len(bin_msg)
print('Message to transmit:', M)
print('Binary message:', bin_msg)
print('Length:', n)

print('*----- FINDING K >= n -----*')
K = ''
while len(K) < n:
    
    # Alice needs to use an n bit key K to encrypt
    # She must first randomly generate two strings of n bits, x & y
    # Here, we use n*2 to account for lost information.
    
    x = random_bits(n * 2)
    y = random_bits(n * 2)
    print('x:', x)
    print('y:', y)
    
    # Bob generates random 1 string of n bits, z
    
    z = random_bits(n * 2)
    print('z:', z)
    
    # Alice prepares 1 of the following qubits:
    #    if xi*yi = 00 -> |xi> =  |0>
    #    if xi*yi = 10 -> |xi> =  |1>
    #    if xi*yi = 01 -> |xi> = H|0> (+)
    #    if xi*yi = 11 -> |xi> = H|1> (-)
    
    # Initialize k (partial K) 
    k = ''
    i = 0
    length = n * 2 - 1
    while length > i:
        if x[i] == '0' and y[i] == '0':
            k += '0'
        elif x[i] == '1' and y[i] == '0':
            k += '1'
        elif x[i] == '0' and y[i] == '1':
            k += '+'
        else:
            k += '-'
        i += 1
    
    # Display value of K (key)
    print('k:', k)
     
    # Alice and Bob discard all qubits for which yi != zi
    
    K = ''
    i = 0
    length = n * 2 - 1
    while length > i:
        if y[i] == z[i]:
            K += k[i]
        i += 1
    K = K[:8]
    
    # If the length of K is less then that of the bits needed to send (n)
    # then this loop continues until that requirement is satisfied

# display new K (key)    
print('K:', K)

# Alice then proceeds to decrypt K herself
#    + -> 0
#    - -> 1

K_Alice = ''
for qubit in K:
    if qubit == '+':
        K_Alice += '0'
    elif qubit == '-':
        K_Alice += '1'
    else:
        K_Alice += qubit

# Alice proceeds to use K (key) to perform gates to each qubit
#    if K[i] = 0 -> do nothing
#    if K[i] = 1 -> x q[i];
#    if K[i] = + -> h q[i];
#    if K[i] = - -> x q[i];
#                -> h q[i];

i = 0
length = n
qasm += '\n// Alice:'
while length > i:
    if K[i] == '1':
        qasm += '\nx q[' + str(i) + '];'
    elif K[i] == '+':
        qasm += '\nh q[' + str(i) + '];'
    elif K[i] == '-':
        qasm += '\nx q[' + str(i) + '];'
        qasm += '\nh q[' + str(i) + '];'
    i += 1

# Bob proceeds to use the same K (key) to perform complementary gates to each qubit
#    if K[i] = 0 -> do nothing
#    if K[i] = 1 -> x q[i];
#    if K[i] = + -> h q[i];
#    if K[i] = - -> x q[i];
#                -> h q[i];
# Bob proceeds to measure each qubit after each operation to uncover K

i = 0
length = n
qasm += '\n// Bob:'
while length > i:
    if K[i] == '+' or K[i] == '-':
        qasm += '\nh q[' + str(i) + '];'
    qasm += '\nmeasure q[' + str(i) + '] -> c[' + str(i) + '];'
    i += 1

# Display OPENQASM code
print('*----- QASM -----*')
print(qasm)

# Initialize API to run state
api.run_experiment(qasm, device, shots, name, timeout)

# Get results and store in codes
try:
    codes = api.get_last_codes()[0]
except:
    codes = api.get_last_codes()[1]
execution_id = ''

# Find execution ID
search_key = 'executions'
for key, executions in codes.items():
    if key == search_key:
        search_key = 'id'
        for key, val in executions[0].items():
            if key == search_key:
                execution_id = val

# Initialize API to retrieve results form exeuction ID
results = api.get_result_from_execution(execution_id)

# Display results
print('*----- RESULTS -----*')
# Display execution ID
print('Execution ID:', execution_id)
print(results)

search_key = 'measure'
for key, label in results.items():
    if key == search_key:
        for key, result in label.items():
            search_key = 'labels'
            if key == search_key:
                # Reverse string to account for 5Q qubit order
                K_Bob = result[0][::-1]
        
print("Alice's key:", K_Alice)
print("Bob's key:", K_Bob)

# Print out results
print('*----- ENCRYPTION -----*')
print('Original ASCII message:', M)
print('Original binary:', bin_msg)
encrypted = encrypt(bin_msg, K_Alice)
print("Encrypted binary using Alice's key:", encrypted)
decrypted = decrypt(encrypted, K_Bob)
print("Decrypted binary using Bob's key:", decrypted)
ascii_msg = text_from_bits(decrypted)
print('Resulting ASCII message:', ascii_msg)
