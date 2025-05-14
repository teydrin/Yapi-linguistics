# Yapi-linguistics

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Yapi Linguistics Builder</title>
  <!-- Import React, ReactDOM and Babel -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/react/18.2.0/umd/react.production.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/18.2.0/umd/react-dom.production.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/7.21.4/babel.min.js"></script>
  <!-- Import Tailwind CSS -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/tailwindcss/2.2.19/tailwind.min.js"></script>
  <style>
    /* Additional custom styles */
    body {
      background-color: #f5f5f5;
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 1rem;
    }
    
    .tree-container {
      max-height: 500px;
      overflow-y: auto;
    }
  </style>
</head>
<body>
  <div id="root"></div>

  <script type="text/babel">
    // React component
    const { useState, useEffect } = React;

    const YapiLinguisticsBuilder = () => {
      // Initial empty tree structure
      const initialTreeData = {
        id: "root",
        name: "Root Family",
        count: "",
        timeperiod: "family",
        children: []
      };

      const [treeData, setTreeData] = useState(initialTreeData);
      const [expanded, setExpanded] = useState({ "root": true });
      const [currentNode, setCurrentNode] = useState(null);
      const [formData, setFormData] = useState({
        name: "",
        type: "language",
        count: ""
      });
      const [jsonView, setJsonView] = useState(false);
      const [projectName, setProjectName] = useState("Sino-Tibetan Languages");

      // Toggle expanded state of a node
      const toggleNode = (nodeId) => {
        setExpanded(prev => ({
          ...prev,
          [nodeId]: !prev[nodeId]
        }));
      };

      // Select a node for editing
      const selectNode = (node) => {
        setCurrentNode(node);
        setFormData({
          name: node.name,
          type: node.timeperiod || "language",
          count: node.count || ""
        });
      };

      // Add a new node
      const addNode = () => {
        if (!currentNode) {
          alert("Please select a parent node first");
          return;
        }
        
        const newId = `node-${Date.now()}`;
        const newNode = {
          id: newId,
          name: formData.name || "New Node",
          count: formData.count,
          timeperiod: formData.type,
          children: []
        };
        
        // Deep clone and add to tree
        const updatedTree = JSON.parse(JSON.stringify(treeData));
        
        // Function to find and update the node
        const findAndAddChild = (node) => {
          if (node.id === currentNode.id) {
            node.children.push(newNode);
            return true;
          }
          
          if (node.children) {
            for (let child of node.children) {
              if (findAndAddChild(child)) {
                return true;
              }
            }
          }
          
          return false;
        };
        
        findAndAddChild(updatedTree);
        setTreeData(updatedTree);
        
        // Expand the parent node
        setExpanded(prev => ({
          ...prev,
          [currentNode.id]: true
        }));
        
        // Reset form
        setFormData({
          name: "",
          type: "language",
          count: ""
        });
      };

      // Update an existing node
      const updateNode = () => {
        if (!currentNode) {
          alert("Please select a node to update");
          return;
        }
        
        // Deep clone and update tree
        const updatedTree = JSON.parse(JSON.stringify(treeData));
        
        // Function to find and update the node
        const findAndUpdateNode = (node) => {
          if (node.id === currentNode.id) {
            node.name = formData.name;
            node.timeperiod = formData.type;
            node.count = formData.count;
            return true;
          }
          
          if (node.children) {
            for (let child of node.children) {
              if (findAndUpdateNode(child)) {
                return true;
              }
            }
          }
          
          return false;
        };
        
        findAndUpdateNode(updatedTree);
        setTreeData(updatedTree);
        
        // Reset form
        setFormData({
          name: "",
          type: "language",
          count: ""
        });
        setCurrentNode(null);
      };

      // Delete a node
      const deleteNode = () => {
        if (!currentNode || currentNode.id === "root") {
          alert("Cannot delete the root node or no node selected");
          return;
        }
        
        // Deep clone and delete from tree
        const updatedTree = JSON.parse(JSON.stringify(treeData));
        
        // Function to find and delete the node
        const findAndDeleteNode = (parent) => {
          if (parent.children) {
            const index = parent.children.findIndex(child => child.id === currentNode.id);
            
            if (index !== -1) {
              parent.children.splice(index, 1);
              return true;
            }
            
            for (let child of parent.children) {
              if (findAndDeleteNode(child)) {
                return true;
              }
            }
          }
          
          return false;
        };
        
        findAndDeleteNode(updatedTree);
        setTreeData(updatedTree);
        
        // Reset form
        setFormData({
          name: "",
          type: "language",
          count: ""
        });
        setCurrentNode(null);
      };

      // Get node type icon/symbol and color
      const getNodeTypeInfo = (node) => {
        // Default values
        let icon = "•";
        let textColor = "text-gray-800";
        let fontWeight = "";
        
        // Check if it's a family
        if (node.timeperiod && node.timeperiod.toLowerCase().includes("family")) {
          icon = "◆";
          textColor = "text-blue-700";
          fontWeight = "font-semibold";
        } 
        // Check if it's an extinct language
        else if (node.timeperiod && node.timeperiod.toLowerCase().includes("extinct")) {
          icon = "†";
          textColor = "text-gray-500";
        }
        // Check if it's a dialect
        else if (node.timeperiod && node.timeperiod.toLowerCase().includes("dialect")) {
          icon = "○";
          textColor = "text-gray-600";
        }
        // Check if it's a spoken language
        else if (node.timeperiod && (
          node.timeperiod.toLowerCase().includes("language") || 
          node.timeperiod.toLowerCase().includes("langauge") || 
          node.timeperiod.toLowerCase().includes("spoken")
        )) {
          icon = "●";
          textColor = "text-green-700";
        }
        
        return { icon, textColor, fontWeight };
      };

      // Draw tree connecting lines
      const renderTreeLines = (level, hasChildren, isLastChild, isExpanded) => {
        if (level === 0) return null;
        
        const lines = [];
        
        // Create vertical lines for all levels except current
        for (let i = 1; i < level; i++) {
          lines.push(
            <div 
              key={`vline-${i}`}
              className="absolute border-l border-gray-300"
              style={{
                left: `${(i * 16) - 8}px`,
                top: '0px',
                height: '100%',
                width: '1px'
              }}
            />
          );
        }
        
        // Create horizontal line to current node
        lines.push(
          <div 
            key="hline"
            className="absolute border-t border-gray-300"
            style={{
              left: `${((level - 1) * 16) - 8}px`,
              top: '10px',
              width: '16px',
              height: '1px'
            }}
          />
        );
        
        return lines;
      };

      // Render a single tree node
      const renderTreeNode = (node, level = 0, isLastChild = false) => {
        const hasChildren = node.children && node.children.length > 0;
        const isExpanded = expanded[node.id] || false;
        const { icon, textColor, fontWeight } = getNodeTypeInfo(node);
        const isSelected = currentNode && currentNode.id === node.id;
        
        return (
          <div key={node.id} className="relative">
            {/* Node header with connector lines */}
            <div className="relative">
              {level > 0 && renderTreeLines(level, hasChildren, isLastChild, isExpanded)}
              
              <div 
                className={`flex items-center py-1.5 relative hover:bg-gray-50 rounded ${isSelected ? 'bg-blue-50' : ''}`}
                style={{ marginLeft: level * 16 + 'px' }}
              >
                {/* Toggle icon or spacer */}
                <div 
                  className={`flex items-center justify-center w-5 h-5 ${hasChildren ? 'cursor-pointer text-gray-500' : 'text-transparent'}`}
                  onClick={() => hasChildren && toggleNode(node.id)}
                >
                  {hasChildren ? (
                    isExpanded ? '▼' : '►'
                  ) : (
                    '►'
                  )}
                </div>
                
                {/* Node icon + name */}
                <div 
                  className="flex items-baseline cursor-pointer"
                  onClick={() => selectNode(node)}
                >
                  <span 
                    className={`mr-1.5 ${textColor}`}
                    title={node.timeperiod}
                  >
                    {icon}
                  </span>
                  <span 
                    className={`${textColor} ${fontWeight} hover:underline`}
                  >
                    {node.name.trim()}
                  </span>
                  
                  {/* Language count if available */}
                  {node.count && node.count.trim() !== "" && (
                    <span className="text-gray-500 text-xs ml-1.5">[{node.count}]</span>
                  )}
                </div>
              </div>
            </div>
            
            {/* Render children if expanded */}
            {hasChildren && isExpanded && (
              <div>
                {node.children.map((child, idx) => 
                  renderTreeNode(
                    child, 
                    level + 1, 
                    idx === node.children.length - 1
                  )
                )}
              </div>
            )}
          </div>
        );
      };

      // Export tree data as JSON
      const exportJson = () => {
        const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(treeData, null, 2));
        const downloadAnchorNode = document.createElement('a');
        downloadAnchorNode.setAttribute("href", dataStr);
        downloadAnchorNode.setAttribute("download", "yapi_linguistics_data.json");
        document.body.appendChild(downloadAnchorNode);
        downloadAnchorNode.click();
        downloadAnchorNode.remove();
      };

      // Import JSON data
      const importJson = (event) => {
        const file = event.target.files[0];
        if (file) {
          const reader = new FileReader();
          reader.onload = (e) => {
            try {
              const jsonData = JSON.parse(e.target.result);
              setTreeData(jsonData);
              setExpanded({ "root": true });
              setCurrentNode(null);
              setFormData({
                name: "",
                type: "language",
                count: ""
              });
            } catch (error) {
              alert("Invalid JSON file");
            }
          };
          reader.readAsText(file);
        }
      };

      // Generate legend item
      const LegendItem = ({ symbol, color, label }) => (
        <div className="flex items-center mr-4 mb-1">
          <span className={`${color} mr-1.5 font-bold`}>{symbol}</span>
          <span className="text-xs text-gray-600">{label}</span>
        </div>
      );

      // Create a new project from scratch
      const newProject = () => {
        if (confirm("This will clear your current work. Continue?")) {
          setTreeData({
            id: "root",
            name: "Root Family",
            count: "",
            timeperiod: "family",
            children: []
          });
          setExpanded({ "root": true });
          setCurrentNode(null);
          setFormData({
            name: "",
            type: "language",
            count: ""
          });
          setProjectName("New Project");
        }
      };

      return (
        <div className="font-sans bg-white shadow-md rounded-md max-w-6xl mx-auto border border-gray-200">
          {/* Top navigation bar */}
          <div className="bg-emerald-700 text-white px-4 py-2 flex justify-between items-center">
            <div className="flex items-center">
              <span className="font-bold text-lg">Yapi Linguistics</span>
              <span className="text-xs ml-2 bg-emerald-600 px-2 py-0.5 rounded">Builder 1.0</span>
            </div>
            <div className="flex items-center space-x-4 text-sm">
              <button 
                onClick={newProject} 
                className="hover:underline"
              >
                New Project
              </button>
              <button
                onClick={() => setJsonView(!jsonView)}
                className="hover:underline"
              >
                {jsonView ? "Tree View" : "JSON View"}
              </button>
              <button
                onClick={exportJson}
                className="hover:underline"
              >
                Export
              </button>
              <label className="hover:underline cursor-pointer">
                Import
                <input
                  type="file"
                  accept=".json"
                  onChange={importJson}
                  className="hidden"
                />
              </label>
            </div>
          </div>

          {/* Project title */}
          <div className="bg-gray-100 p-3 border-b border-gray-300">
            <div className="flex items-center">
              <span className="font-medium mr-2">Project:</span>
              <input
                type="text"
                value={projectName}
                onChange={(e) => setProjectName(e.target.value)}
                className="px-2 py-1 border border-gray-300 rounded focus:outline-none focus:ring-1 focus:ring-emerald-500 text-sm flex-grow"
              />
            </div>
          </div>
          
          {/* Main content area */}
          <div className="flex flex-col md:flex-row">
            {/* Tree view or JSON view */}
            <div className="w-full md:w-2/3 p-4 border-r border-gray-200">
              {jsonView ? (
                <div className="h-96 overflow-auto">
                  <pre className="text-xs bg-gray-50 p-4 border border-gray-200 rounded">
                    {JSON.stringify(treeData, null, 2)}
                  </pre>
                </div>
              ) : (
                <>
                  {/* Legend */}
                  <div className="mb-4">
                    <div className="p-2 bg-gray-50 border border-gray-200 rounded text-sm">
                      <div className="flex flex-wrap">
                        <LegendItem symbol="◆" color="text-blue-700" label="Family" />
                        <LegendItem symbol="●" color="text-green-700" label="Living Language" />
                        <LegendItem symbol="†" color="text-gray-500" label="Extinct Language" />
                        <LegendItem symbol="○" color="text-gray-600" label="Dialect" />
                      </div>
                    </div>
                  </div>
                  
                  {/* Tree container */}
                  <div className="border border-gray-200 rounded bg-white mb-4 p-1 h-96 overflow-auto">
                    <div className="relative">
                      {renderTreeNode(treeData, 0)}
                    </div>
                  </div>
                </>
              )}
            </div>
            
            {/* Node editor */}
            <div className="w-full md:w-1/3 p-4 bg-gray-50">
              <h2 className="text-lg font-medium mb-4">
                {currentNode ? "Edit Node" : "Add New Node"}
              </h2>
              
              {currentNode && (
                <div className="mb-4 p-2 bg-blue-50 border border-blue-200 rounded text-sm">
                  <div className="font-medium">Selected:</div>
                  <div>{currentNode.name}</div>
                </div>
              )}
              
              <div className="space-y-4">
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Name
                  </label>
                  <input
                    type="text"
                    value={formData.name}
                    onChange={(e) => setFormData({...formData, name: e.target.value})}
                    className="w-full px-3 py-2 border border-gray-300 rounded focus:outline-none focus:ring-1 focus:ring-emerald-500"
                    placeholder="Language or family name"
                  />
                </div>
                
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Type
                  </label>
                  <select
                    value={formData.type}
                    onChange={(e) => setFormData({...formData, type: e.target.value})}
                    className="w-full px-3 py-2 border border-gray-300 rounded focus:outline-none focus:ring-1 focus:ring-emerald-500"
                  >
                    <option value="family">Family</option>
                    <option value="language">Living Language</option>
                    <option value="extinct">Extinct Language</option>
                    <option value="dialect">Dialect</option>
                  </select>
                </div>
                
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Count (optional)
                  </label>
                  <input
                    type="text"
                    value={formData.count}
                    onChange={(e) => setFormData({...formData, count: e.target.value})}
                    className="w-full px-3 py-2 border border-gray-300 rounded focus:outline-none focus:ring-1 focus:ring-emerald-500"
                    placeholder="Number of languages"
                  />
                </div>
                
                <div className="flex space-x-2 pt-2">
                  <button
                    onClick={addNode}
                    className="bg-emerald-600 text-white px-4 py-2 rounded hover:bg-emerald-700 text-sm flex-1"
                  >
                    Add as Child
                  </button>
                  
                  {currentNode && (
                    <>
                      <button
                        onClick={updateNode}
                        className="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700 text-sm flex-1"
                      >
                        Update
                      </button>
                      
                      <button
                        onClick={deleteNode}
                        className="bg-red-600 text-white px-4 py-2 rounded hover:bg-red-700 text-sm"
                      >
                        Delete
                      </button>
                    </>
                  )}
                </div>
              </div>
              
              <div className="mt-8">
                <h3 className="text-sm font-medium mb-2">How to use:</h3>
                <ol className="text-xs text-gray-600 list-decimal pl-4 space-y-1">
                  <li>Click on a node to select it as the parent</li>
                  <li>Add details in the form and click "Add as Child"</li>
                  <li>To edit a node, select it and update the form</li>
                  <li>Use Export to save your work as JSON</li>
                  <li>Use Import to load previously saved work</li>
                </ol>
              </div>
            </div>
          </div>
          
          {/* Footer */}
          <div className="bg-gray-100 p-3 border-t border-gray-300 text-xs text-gray-600">
            <div className="flex flex-col md:flex-row md:justify-between">
              <div>© 2025 Yapi Linguistics • Open Source Linguistic Classification Builder</div>
              <div className="flex space-x-4 mt-1 md:mt-0">
                <a href="#" className="text-emerald-600 hover:underline">GitHub</a>
                <a href="#" className="text-emerald-600 hover:underline">Documentation</a>
                <a href="#" className="text-emerald-600 hover:underline">License</a>
              </div>
            </div>
          </div>
        </div>
      );
    };

    // Render the component
    const root = ReactDOM.createRoot(document.getElementById('root'));
    root.render(<YapiLinguisticsBuilder />);
  </script>
</body>
</html>
