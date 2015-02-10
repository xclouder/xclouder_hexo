title: 反向传播神经网络算法——ruby版的XOR实例
id: 54
categories:
  - 未分类
date: 2013-03-16 13:40:48
tags:
---

`
srand 0
def myrand(a,b)
	target = (b - a) * rand + a
	return target
end

#神经结点
class Node
	def initialize(name)
		@name = name
		@value = 0
		@outputs = nil
		@inputs = nil
		@delta = 0
	end

	#绑定一个下一层的结点
	def connectNextLayerNode(node)
		connWeight = NodeConnectionWeight.new(myrand(-2, 2))
		connection = NodeConnection.new(connWeight)
		connection.toNode = node
		connectionBack = NodeConnection.new(connWeight)
		connectionBack.toNode = self

		self.addOutputConnect(connection)
		node.addInputConnect(connectionBack)
	end

	def addOutputConnect(connect)
		if self.outputs == nil
			self.outputs = [connect]
		else
			self.outputs << connect
		end
	end

	def addInputConnect(connect)
		if self.inputs == nil
			self.inputs = [connect]
		else
			self.inputs << connect
		end
	end

	def updateValue
		if self.inputs == nil
			puts 'WARN:no inputs, discard updateValue operation'
			return
		else
			tmpVal = 0
			self.inputs.each { |conn|
				tmpVal += conn.toNode.value * conn.weight.value
			}
			self.value = tmpVal
		end
	end

	def printConnectionNames
		if (self.inputs)
			puts "========================================"
			puts "[#{self.name}]'s input connections:"
			self.inputs.each { |conn|
				print "#{conn.toNode.name}:#{conn.weight.value} "
			}
			puts 

		end
		if (self.outputs)
			puts "[#{self.name}]'s output connections:"
			self.outputs.each { |conn|
				print "#{conn.toNode.name}:#{conn.weight.value} "
			}			
			puts
		end
	end

	attr_accessor :value, :outputs, :inputs, :name, :delta
	protected :addOutputConnect, :addInputConnect
end

#结点连接的权重
class NodeConnectionWeight
	def initialize(val)
		@value = val
	end

	attr_accessor :value
end

#神经结点的连接
class NodeConnection
	def initialize (w)
		@weight = w
		@previousWeight = NodeConnectionWeight.new(0)
		@value = 0
		@toNode = nil
	end
	attr_accessor :weight,:value, :toNode, :previousWeight
end

#神经网络的一层
class BPNetworkLayer
	def initialize(name, numOfNodes)
		@name = name
		@nodes = []
		0.upto(numOfNodes - 1){ |i|
			@nodes << Node.new(name + "-" + i.to_s)
		}
	end

	attr_accessor :nodes, :name
end

class BPNetworkInputLayer < BPNetworkLayer
	def initialize(name, numOfNodes)
		super(name, numOfNodes)

		biasNode = Node.new(name + "-bias node")
		biasNode.value = 1.0
		@nodes << biasNode
	end
end

#前馈神经网络
class BPNetwork
	def initialize
		@layers = nil
		@speed = 0.01
		@momentum = 0.1
		@outputLayer = nil;
		@inputLayer = nil
		@hasBuildConnection = false
	end

	def addLayer(layer)
		if self.layers == nil
			self.layers = [layer]
		else
			self.layers << layer
		end
	end

	def train(data)
		if !@hasBuildConnection
			self.buildConnection

			#打印网络中结点的连接情况
			self.printNetworkConnection
		end

		inputs = data[0]
		targetOutputs = data[1]

		#TODO注意越界情况
		inputNodes = self.inputLayer.nodes
		inputs.each_index { |i|
			inputNodes[i].value = inputs[i]
		}

		#计算输出值
		self.update
		# puts "outputs:#{self.outputLayer.nodes.inspect}"

		#err
		err = self.backPropagate(targetOutputs)
		return err
	end

	def update
		#通过输入，计算网络输出值
		layersCount = self.layers.length
		1.upto(layersCount - 1) { |i|
			currentLayerNodes = self.layers[i].nodes
			currentLayerNodes.each { |node|
				node.updateValue

				node.value = self.sigmoid(node.value)
			}
		}
	end

	def backPropagate(targets)
		outputNotes = self.outputLayer.nodes
		if targets.length != outputNotes.length
			puts "wrong number of target values"
			return 0
		end

		#计算输出单元的误差项delta
		err = 0
		outputNotes.each_index { |i|
			outputNode = outputNotes[i]
			outputVal = outputNode.value
			# outputNode.delta = (targets[i] - outputVal) * outputVal * (1 - outputVal)
			outputNode.delta = dsigmoid(outputNode.value) * (targets[i] - outputVal)

			err += 0.5 * (targets[i] - outputVal) ** 2
		}

		layersCount = self.layers.count
		# puts "=========="
		#这里只适用于层数大于1的情况
		(layersCount - 2).downto(1) { |i|
			currentLayerNodes = self.layers[i].nodes
			currentLayerNodes.each { |node|
				nextNodesAvgDelta = 0.0
				node.outputs.each { |conn|
					# puts conn.toNode.name
					nextNodesAvgDelta += conn.toNode.delta * conn.weight.value
					# puts "fromNode:#{node.name} toNode:#{conn.toNode.name} weoght:#{conn.weight.value}"
				}
				# node.delta = node.value * (1 - node.value) * nextNodesAvgDelta
				node.delta = self.dsigmoid(node.value) * nextNodesAvgDelta
			}
		}

		(layersCount - 1).downto(1) { |i|
			currentLayerNodes = self.layers[i].nodes
			currentLayerNodes.each { |node|				

				if node.inputs != nil
					node.inputs.each { |conn|
						change = node.delta * conn.toNode.value
						delta = self.speed * change + self.momentum * conn.previousWeight.value
						conn.weight.value += delta 
						conn.previousWeight.value = change
						# puts "fromNode:#{conn.toNode.name} toNode:#{node.name} weight:#{conn.weight.value} |delta:#{delta}"
					}
				end
			}
		}

		return err
	end

	def getResult(inputData)
		inputNodes = self.inputLayer.nodes
		inputData.each_index { |i|
			inputNodes[i].value = inputData[i]
		}

		self.update

		outputNode = self.outputLayer.nodes
		output = []
		outputNode.each { |node|
			output << node.value
		}

		return output
	end

	#protected methods
	def buildConnection
		@hasBuildConnection = true

		if self.layers == nil
			puts 'nodes empty, exit'
			return false
		end

		#build connection
		@outputLayer = self.layers.last
		@inputLayer = self.layers.first
		# iterate the layers
		len = self.layers.length
		0.upto(len - 1) { |i|
			if i < (len - 1)
				layer = self.layers[i]
				layerNext = self.layers[i + 1]
				self.connect(layer, layerNext)
			end
		}
		return true
	end

	def connect(layer, layerNext)
		layer.nodes.each { |node|
			layerNext.nodes.each { |nextNode|
				node.connectNextLayerNode(nextNode)
			}
		}
	end

	#sigmoid函数采用了"y = 1 / (1 + e ^ -x)"
	def sigmoid(x)
		# return 1.0 / (1.0 + Math.exp(-x))
		return Math.tanh(x)
	end

	def dsigmoid(x)
		# return (1.0 - x) * x
		return 1.0 - x * x
	end

	def printNetworkConnection

		self.layers.each { |layer|
			layer.nodes.each { |node|
				node.printConnectionNames
			}
		}
	end

	protected :buildConnection, :connect
	attr_accessor :layers, :outputLayer, :inputLayer, :speed, :momentum
end

inputLayer = BPNetworkInputLayer.new("Input", 2)
inputLayer2 = BPNetworkLayer.new("Hidden1",2)
inputLayer3 = BPNetworkLayer.new("Output",1)

network = BPNetwork.new()

network.speed = 0.5
network.momentum = 0.2
network.addLayer(inputLayer)
network.addLayer(inputLayer2)
network.addLayer(inputLayer3)

1.upto(10000) { |i|
	err = 0
	err += network.train([[0,0],[0]])
	err += network.train([[0,1],[1]])
	err += network.train([[1,0],[1]])
	err += network.train([[1,1],[0]])

	if (i % 100 == 0)
		puts "error #{err}"
	end
}

# network.printNetworkConnection

puts network.getResult([1,0])
puts network.getResult([1,1])
puts network.getResult([0,1])
puts network.getResult([0,0])

`