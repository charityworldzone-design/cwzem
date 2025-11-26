export default {
  async fetch(request, env) {
    const body = await request.json();
    const { messages } = body;

    // pick last 5 messages for context
    const lastMessages = messages.slice(-5);

    const prompt = lastMessages.map(m => `${m.role === 'user' ? "User" : "CWZ"}: ${m.content}`).join("\n") + "\nCWZ:";

    const response = await fetch("https://api-inference.huggingface.co/models/Qwen/Qwen2.5-1.5B-Instruct", {
      method: "POST",
      headers: {
        "Authorization": `Bearer ${env.HF_TOKEN}`,
        "Content-Type": "application/json"
      },
      body: JSON.stringify({
        inputs: prompt,
        parameters: { max_new_tokens: 200 }
      })
    });

    const data = await response.json();
    const reply = data.generated_text || "Sorry, CWZ couldn't respond.";

    return new Response(JSON.stringify({ reply }), {
      headers: { "Content-Type": "application/json" },
    });
  }
}