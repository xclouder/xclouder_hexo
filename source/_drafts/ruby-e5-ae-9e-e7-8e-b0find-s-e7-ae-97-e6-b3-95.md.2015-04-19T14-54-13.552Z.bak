title: ruby实现Find-S算法
id: 27
categories:
  - ML
date: 2013-01-22 02:05:08
tags:
---

实现FIND-S算法。首先，验证它可成功地产生2.4节中EnjoySport各个步骤的结果。然后使用这一程序，研究为了学习到确切的目标概念所需的随机训练样例数目。实现一训练样例生成器来生成这些随机的实例，再用下面的目标概念产生分类结果：

<Sunny, Warm, ?, ?, ?, ?>

试用随机产生样例训练你的FIND-S算法并测量需要多少样例才能使程序的假设与目标概念相等。能否预测所需的平均样例数目？运行该实验20次并报告所需样例的平均数。这一数目会怎样随着目标概念中的“？”数目而变动？以及它会怎样随着实例或假设中属性的数目而变动？
`
class FindS
	def initialize
		@learnedConcept = {
			"Sky" => nil,
			"AirTemp" => nil,
			"Humidity" => nil,
			"Wind" => nil,
			"Water" => nil,
			"Forecast" => nil
		}
	end

	def learn(anEx,isPass)
		if isPass
			@learnedConcept.each do |key, value|
				if value == nil
					@learnedConcept[key] = anEx[key]
				else
					if @learnedConcept[key] != anEx[key]
						@learnedConcept.delete(key)
					end
				end
			end
		else
			# do nothing
		end
	end

	attr_reader:learnedConcept
end

class ExampleGenerator
	@@factors = {
		"Sky" => ["Sunny", "Cloudy", "Rainy"],
		"AirTemp" => ["Warm", "Cold"],
		"Humidity" => ["Normal", "High"],
		"Wind" => ["Strong", "Weak"],
		"Water" => ["Warm", "Cool"],
		"Forecast" => ["Same", "Change"]
	}

	def initialize
		@generatedHash = []
	end

# private methods
	def record(ex)
		#properties name len are different, use it
		hash = 0
		ex.each { |key, value|
			len = key.length
			valueArr = @@factors[key]
			index = valueArr.index(value)

			flag = (10 ** len) * index
			if flag > 0
				hash += flag
			end
		}

		if @generatedHash.include?(hash)
			# puts "existing example:" + ex.to_s
			return false
		else
			@generatedHash << hash
			return true
		end
	end

# public methods
	def generate

		aExample = {}
		@@factors.each {|key, value|
			len = value.length
			randomIndex = rand(len)

			aExample[key] = value[randomIndex]
		}

		return aExample
	end

	def generateDifferently
		begin
		   ex = self.generate
		end while !self.record(ex)
		return ex
	end
end

class Classifier
	def initialize
		@targetConcept = {"Sky" => "Sunny", "AirTemp" => "Warm"}
	end

	def isConceptRight?(aConcept)

		if @targetConcept == aConcept
			return true
		else
			return false
		end
	end

	def isExamplePass?(anExample)
		@targetConcept.each{|key, value|
			return false if value != anExample[key]
		}
		return true
	end
end

totalExamples = 0
trycount = 200
for i in 1..trycount
	generator = ExampleGenerator.new
	classifier = Classifier.new
	finder = FindS.new

	exNum = 0
	while !classifier.isConceptRight?(finder.learnedConcept)
		ex = generator.generateDifferently
		isPass = classifier.isExamplePass?(ex)
		finder.learn(ex, isPass)
		exNum += 1;
	end

	puts "learnedConcept:"+finder.learnedConcept.inspect
	puts "examples:" + exNum.to_s
	totalExamples += exNum
end

puts "avg examples:#{totalExamples / trycount}"

`