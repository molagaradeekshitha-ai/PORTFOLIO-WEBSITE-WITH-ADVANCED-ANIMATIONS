# PORTFOLIO-WEBSITE-WITH-ADVANCED-ANIMATIONS
import React, { useEffect, useMemo, useRef } from "react";
import { Canvas, useFrame } from "@react-three/fiber";
import { Points, PointMaterial, OrbitControls } from "@react-three/drei";
import * as THREE from "three";
import { gsap } from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Mail, Github, Linkedin, ArrowDown, ExternalLink, Code2, Rocket } from "lucide-react";

// Register GSAP plugin
if (typeof window !== "undefined" && gsap && ScrollTrigger) {
  gsap.registerPlugin(ScrollTrigger);
}

// ===== 3D Background: Animated Starfield =====
function Starfield({ count = 4000 }) {
  const ref = useRef();
  const sphere = useMemo(() => new Float32Array(count * 3), [count]);

  useMemo(() => {
    const radius = 1.5;
    for (let i = 0; i < count; i++) {
      const i3 = i * 3;
      const r = radius * Math.cbrt(Math.random());
      const theta = Math.random() * 2 * Math.PI;
      const phi = Math.acos(2 * Math.random() - 1);
      sphere[i3] = r * Math.sin(phi) * Math.cos(theta);
      sphere[i3 + 1] = r * Math.sin(phi) * Math.sin(theta);
      sphere[i3 + 2] = r * Math.cos(phi);
    }
  }, [count, sphere]);

  useFrame((state, delta) => {
    if (!ref.current) return;
    ref.current.rotation.x -= delta * 0.015;
    ref.current.rotation.y -= delta * 0.01;
  });

  return (
    <group rotation={[0, 0, Math.PI / 8]}>
      <Points ref={ref} positions={sphere} stride={3} frustumCulled>
        <PointMaterial transparent size={0.01} depthWrite={false} sizeAttenuation />
      </Points>
    </group>
  );
}

// ===== Hook: GSAP Section Animations =====
function useSectionAnimations() {
  const refs = {
    nav: useRef(null),
    hero: useRef(null),
    about: useRef(null),
    projects: useRef(null),
    skills: useRef(null),
    contact: useRef(null),
  };

  useEffect(() => {
    if (!refs.hero.current) return;

    // Nav shrink + background on scroll
    ScrollTrigger.create({
      trigger: refs.hero.current,
      start: "top top",
      end: "bottom top",
      onUpdate: (self) => {
        const nav = refs.nav.current;
        if (!nav) return;
        const scrolled = self.progress > 0.05;
        nav.classList.toggle("backdrop-blur", scrolled);
        nav.classList.toggle("bg-white/60", scrolled);
        nav.classList.toggle("shadow", scrolled);
        nav.classList.toggle("py-6", !scrolled);
        nav.classList.toggle("py-3", scrolled);
      },
    });

    const fadeUp = (el) => {
      gsap.fromTo(
        el.querySelectorAll("[data-animate='in']"),
        { y: 24, opacity: 0 },
        {
          y: 0,
          opacity: 1,
          duration: 0.9,
          stagger: 0.08,
          ease: "power3.out",
          scrollTrigger: {
            trigger: el,
            start: "top 80%",
          },
        }
      );
    };

    [refs.about, refs.projects, refs.skills, refs.contact].forEach((r) => {
      if (r.current) fadeUp(r.current);
    });

    // Hero title intro
    if (refs.hero.current) {
      const tl = gsap.timeline();
      tl.fromTo(
        refs.hero.current.querySelectorAll("[data-hero='title']"),
        { y: 30, opacity: 0 },
        { y: 0, opacity: 1, duration: 1.1, ease: "power3.out", stagger: 0.07 }
      ).fromTo(
        refs.hero.current.querySelectorAll("[data-hero='cta']"),
        { y: 20, opacity: 0 },
        { y: 0, opacity: 1, duration: 0.8, ease: "power3.out" },
        "-=0.4"
      );
    }
  }, []);

  return refs;
}

// ===== Utility: Smooth scroll =====
function scrollToId(id) {
  const el = document.getElementById(id);
  if (el) el.scrollIntoView({ behavior: "smooth", block: "start" });
}

// ===== Main Component =====
export default function PortfolioSite() {
  const refs = useSectionAnimations();

  return (
    <div className="min-h-screen bg-gradient-to-b from-slate-50 via-white to-white text-slate-900">
      {/* Nav */}
      <header ref={refs.nav} className="fixed left-0 right-0 top-0 z-50 py-6 transition-all">
        <div className="mx-auto flex max-w-6xl items-center justify-between px-6">
          <div className="flex items-center gap-2">
            <div className="grid h-9 w-9 place-items-center rounded-2xl bg-slate-900 text-white">
              <Rocket className="h-5 w-5" />
            </div>
            <span className="font-semibold tracking-tight">Your Name</span>
          </div>
          <nav className="hidden gap-6 md:flex">
            {[
              ["About", "about"],
              ["Projects", "projects"],
              ["Skills", "skills"],
              ["Contact", "contact"],
            ].map(([label, id]) => (
              <button
                key={id}
                onClick={() => scrollToId(id)}
                className="text-sm font-medium text-slate-700 transition hover:text-slate-900"
              >
                {label}
              </button>
            ))}
          </nav>
          <div className="flex items-center gap-2">
            <a href="#contact" onClick={(e) => { e.preventDefault(); scrollToId("contact"); }}>
              <Button className="rounded-2xl">Hire Me</Button>
            </a>
          </div>
        </div>
      </header>

      {/* Hero */}
      <section ref={refs.hero} id="hero" className="relative flex min-h-screen items-center overflow-hidden pt-24">
        <div className="pointer-events-none absolute inset-0 -z-10">
          <Canvas camera={{ position: [0, 0, 2.4], fov: 60 }}>
            <ambientLight intensity={0.4} />
            <Starfield count={6000} />
            <OrbitControls enableZoom={false} enablePan={false} autoRotate autoRotateSpeed={0.3} />
          </Canvas>
          <div className="absolute inset-0 bg-gradient-to-b from-white/60 via-white/10 to-white"></div>
        </div>

        <div className="mx-auto grid max-w-6xl grid-cols-1 items-center gap-10 px-6 md:grid-cols-2">
          <div>
            <h1 className="mb-4 text-4xl font-bold leading-tight tracking-tight md:text-6xl">
              <span data-hero="title" className="block">Hi, I’m <span className="bg-gradient-to-r from-slate-900 to-slate-600 bg-clip-text text-transparent">Your Name</span></span>
              <span data-hero="title" className="block">Front‑End Developer</span>
            </h1>
            <p data-hero="title" className="mb-6 max-w-prose text-slate-600">
              I craft immersive web experiences with modern stacks. I love building performant UIs, silky‑smooth animations, and delightful interactions.
            </p>
            <div className="flex flex-wrap gap-3" data-hero="cta">
              <Button onClick={() => scrollToId("projects")} className="rounded-2xl">
                View Projects
              </Button>
              <Button variant="outline" onClick={() => scrollToId("about")} className="rounded-2xl">
                Learn More
              </Button>
            </div>
            <div className="mt-10 flex items-center gap-4 opacity-80" data-hero="cta">
              <a href="https://github.com/" target="_blank" rel="noreferrer" className="transition hover:opacity-100">
                <Github />
              </a>
              <a href="https://linkedin.com/" target="_blank" rel="noreferrer" className="transition hover:opacity-100">
                <Linkedin />
              </a>
              <a href="#contact" onClick={(e) => { e.preventDefault(); scrollToId("contact"); }} className="transition hover:opacity-100">
                <Mail />
              </a>
            </div>
          </div>

          <div className="relative mx-auto aspect-square w-full max-w-md overflow-hidden rounded-3xl shadow-2xl ring-1 ring-black/5">
            {/* Replace the img src with your photo */}
            <img
              src="https://images.unsplash.com/photo-1544006659-f0b21884ce1d?q=80&w=1200&auto=format&fit=crop"
              alt="Portrait"
              className="h-full w-full object-cover"
            />
            <div className="pointer-events-none absolute inset-0 bg-gradient-to-tr from-white/0 via-white/0 to-white/40" />
          </div>
        </div>

        <div className="absolute bottom-6 left-1/2 -translate-x-1/2 animate-bounce text-slate-500">
          <ArrowDown />
        </div>
      </section>

      {/* About */}
      <section id="about" ref={refs.about} className="mx-auto max-w-6xl scroll-mt-24 px-6 py-28">
        <div className="mx-auto max-w-3xl text-center">
          <h2 className="mb-3 text-3xl font-semibold tracking-tight">About Me</h2>
          <p className="text-slate-600" data-animate="in">
            I’m a front‑end developer focused on building accessible, fast, and animated interfaces. My toolkit includes React, TypeScript, Tailwind, GSAP, and Three.js.
          </p>
        </div>
        <div className="mt-10 grid gap-6 md:grid-cols-3" data-animate="in">
          {["Performance‑first", "Animation‑savvy", "Pixel‑perfect"].map((title) => (
            <Card key={title} className="rounded-2xl">
              <CardHeader>
                <CardTitle className="flex items-center gap-2"><Code2 className="h-5 w-5" /> {title}</CardTitle>
              </CardHeader>
              <CardContent>
                <p className="text-sm text-slate-600">
                  Lorem ipsum dolor sit amet, consectetur adipiscing elit. Fusce dapibus interdum nunc, vitae fermentum.
                </p>
              </CardContent>
            </Card>
          ))}
        </div>
      </section>

      {/* Projects */}
      <section id="projects" ref={refs.projects} className="bg-slate-50/60 py-28">
        <div className="mx-auto max-w-6xl px-6">
          <div className="mx-auto mb-10 max-w-3xl text-center">
            <h2 className="mb-3 text-3xl font-semibold tracking-tight">Featured Projects</h2>
            <p className="text-slate-600" data-animate="in">Hover to preview and click to view details.</p>
          </div>

          <div className="grid gap-6 md:grid-cols-2" data-animate="in">
            {[
              {
                title: "Interactive 3D Landing Page",
                img: "https://images.unsplash.com/photo-1522071901873-411886a10004?q=80&w=1200&auto=format&fit=crop",
                href: "#",
              },
              {
                title: "GSAP Scroll Animations",
                img: "https://images.unsplash.com/photo-1556157382-97eda2d62296?q=80&w=1200&auto=format&fit=crop",
                href: "#",
              },
              {
                title: "Data Visualization Dashboard",
                img: "https://images.unsplash.com/photo-1498050108023-c5249f4df085?q=80&w=1200&auto=format&fit=crop",
                href: "#",
              },
              {
                title: "E‑commerce UI Kit",
                img: "https://images.unsplash.com/photo-1504639725590-34d0984388bd?q=80&w=1200&auto=format&fit=crop",
                href: "#",
              },
            ].map((p) => (
              <a
                key={p.title}
                href={p.href}
                target={p.href.startsWith("http") ? "_blank" : undefined}
                rel={p.href.startsWith("http") ? "noreferrer" : undefined}
                className="group relative overflow-hidden rounded-3xl ring-1 ring-slate-200"
              >
                <img src={p.img} alt={p.title} className="aspect-[16/10] w-full object-cover transition duration-700 group-hover:scale-105" />
                <div className="absolute inset-0 bg-gradient-to-t from-black/50 via-black/10 to-transparent opacity-60 transition group-hover:opacity-80" />
                <div className="absolute inset-x-0 bottom-0 flex items-center justify-between p-5 text-white">
                  <div>
                    <div className="text-lg font-semibold">{p.title}</div>
                    <div className="text-xs opacity-80">React • GSAP • Three.js</div>
                  </div>
                  <ExternalLink className="h-5 w-5 opacity-80 transition group-hover:translate-x-0.5 group-hover:-translate-y-0.5" />
                </div>
              </a>
            ))}
          </div>
        </div>
      </section>

      {/* Skills */}
      <section id="skills" ref={refs.skills} className="mx-auto max-w-6xl scroll-mt-24 px-6 py-28">
        <div className="mx-auto mb-10 max-w-3xl text-center">
          <h2 className="mb-3 text-3xl font-semibold tracking-tight">Skills</h2>
          <p className="text-slate-600" data-animate="in">A stack I use to ship polished experiences.</p>
        </div>
        <ul className="mx-auto grid max-w-4xl grid-cols-2 gap-3 text-sm md:grid-cols-4" data-animate="in">
          {["React", "TypeScript", "Tailwind CSS", "GSAP", "Three.js", "Next.js", "Node.js", "Framer Motion"].map((s) => (
            <li key={s} className="rounded-xl border border-slate-200 bg-white px-4 py-3 text-center shadow-sm">
              {s}
            </li>
          ))}
        </ul>
      </section>

      {/* Contact */}
      <section id="contact" ref={refs.contact} className="bg-slate-50/60 py-28">
        <div className="mx-auto max-w-6xl px-6">
          <div className="mx-auto mb-10 max-w-3xl text-center">
            <h2 className="mb-3 text-3xl font-semibold tracking-tight">Let’s work together</h2>
            <p className="text-slate-600" data-animate="in">
              Have a project in mind? Drop a message and I’ll get back to you.
            </p>
          </div>
          <form className="mx-auto grid max-w-3xl gap-4 rounded-3xl bg-white p-6 shadow">
            <div className="grid gap-2 md:grid-cols-2">
              <input className="rounded-2xl border border-slate-200 p-3 outline-none focus:ring-2 focus:ring-slate-300" placeholder="Your name" />
              <input className="rounded-2xl border border-slate-200 p-3 outline-none focus:ring-2 focus:ring-slate-300" placeholder="Email" type="email" />
            </div>
            <input className="rounded-2xl border border-slate-200 p-3 outline-none focus:ring-2 focus:ring-slate-300" placeholder="Subject" />
            <textarea className="min-h-[140px] rounded-2xl border border-slate-200 p-3 outline-none focus:ring-2 focus:ring-slate-300" placeholder="Message" />
            <div className="flex justify-end">
              <Button type="button" className="rounded-2xl">Send Message</Button>
            </div>
          </form>
        </div>
      </section>

      {/* Footer */}
      <footer className="py-10 text-center text-sm text-slate-500">
        © {new Date().getFullYear()} Your Name. Built with React, GSAP & Three.js.
      </footer>
    </div>
  );
}
